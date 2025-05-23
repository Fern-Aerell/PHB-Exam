<script setup lang="ts">
import { useStore } from '@/store';
import { usePage } from '@inertiajs/vue3';
import { onMounted, onUnmounted, ref, watchEffect } from 'vue';

// #ENUM
enum Status {
    None = 0,
    StartingSoon = 2,
    OnGoing = 3,
    Holiday = 4,
    Finished = 5,
    FutureExam = 6
}

// #VARIABLES
const examStatus = ref<Status>(Status.None);
const examDate = ref<string | null>(null);
const examTimeRemaining = ref<string | null>(null);
const store = useStore();

let intervalId: NodeJS.Timeout | null = null;

function set(
    exam_date_start: string = usePage().props.config.exam_date_start,
    exam_date_end: string = usePage().props.config.exam_date_end,
    holiday_date: string | null = usePage().props.config.holiday_date,
    exam_time_start: string = usePage().props.config.exam_time_start,
    exam_time_end: string = usePage().props.config.exam_time_end
) {
    const startDate = new Date(exam_date_start);
    const endDate = new Date(exam_date_end);
    const currentDate = new Date();

    // Reset time part for date comparison
    startDate.setHours(0, 0, 0, 0);
    endDate.setHours(23, 59, 59, 999);
    currentDate.setHours(0, 0, 0, 0);

    const holidayDates = holiday_date ? holiday_date.split(',').map(day => parseInt(day.trim())) : [];

    const isHoliday = (date: Date): boolean => {
        return holidayDates.includes(date.getDate());
    };

    const formatDate = (date: Date): string => {
        return `${date.getDate()} ${date.toLocaleString('id-ID', { month: 'short' })} ${date.getFullYear()}, Jam ${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}`;
    };

    if (currentDate < startDate) {
        examStatus.value = Status.FutureExam;
        const [startHour, startMinute] = exam_time_start.split(':').map(Number);
        startDate.setHours(startHour, startMinute, 0);
        examDate.value = formatDate(startDate);
        store.setIsExamTime(false);
        return;
    }

    if (currentDate < startDate || currentDate > endDate) {
        examStatus.value = Status.None;
        store.setIsExamTime(false);
        return;
    }

    if (isHoliday(currentDate)) {
        examStatus.value = Status.Holiday;
        store.setIsExamTime(false);
        return;
    }

    const [startHour, startMinute] = exam_time_start.split(':').map(Number);
    const [endHour, endMinute] = exam_time_end.split(':').map(Number);

    const examStartTime = new Date(currentDate);
    examStartTime.setHours(startHour, startMinute, 0);

    const examEndTime = new Date(currentDate);
    examEndTime.setHours(endHour, endMinute, 0);

    const calculateTimeRemaining = (target: Date): string => {
        const diff = target.getTime() - new Date().getTime();
        if (diff <= 0) return "0 detik";

        const seconds = Math.floor(diff / 1000);
        const minutes = Math.floor(seconds / 60);
        const hours = Math.floor(minutes / 60);

        if (hours > 0) return `${hours} jam ${minutes % 60} menit lagi`;
        if (minutes > 0) return `${minutes} menit lagi`;
        return `${seconds} detik lagi`;
    };

    const now = new Date();

    if (now < examStartTime) {
        examStatus.value = Status.StartingSoon;
        examDate.value = formatDate(examStartTime);
        examTimeRemaining.value = calculateTimeRemaining(examStartTime);
        store.setIsExamTime(false);
    } else if (now >= examStartTime && now < examEndTime) {
        examStatus.value = Status.OnGoing;
        examDate.value = formatDate(examEndTime);
        examTimeRemaining.value = calculateTimeRemaining(examEndTime);
        store.setIsExamTime(true);
    } else {
        examStatus.value = Status.Finished;
        store.setIsExamTime(false);
    }
}

function updateExamTimeCard(event: {
    exam_date_start: string,
    exam_date_end: string,
    holiday_date: string | null,
    exam_time_start: string,
    exam_time_end: string
}) {
    usePage().props.config.exam_date_start = event.exam_date_start;
    usePage().props.config.exam_date_end = event.exam_date_end;
    usePage().props.config.holiday_date = event.holiday_date != null ? event.holiday_date : '';
    usePage().props.config.exam_time_start = event.exam_time_start;
    usePage().props.config.exam_time_end = event.exam_time_end;
}

// #LIFECYCLE
onMounted(() => {
    window.Echo.channel('examTimeCard').listen('UpdateExamTimeCard', updateExamTimeCard);

    set();
    intervalId = setInterval(set, 1000);
});

onUnmounted(() => {
    window.Echo.channel('examTimeCard').stopListening('UpdateExamTimeCard', updateExamTimeCard);

    if (intervalId !== null) {
        clearInterval(intervalId);
    }
});

</script>

<template>
    <div :class="[
        'rounded-[10px] p-[10px]',
        {
            'bg-[#F1E07F]': examStatus === Status.StartingSoon,
            'bg-[#F1C57F]': examStatus === Status.OnGoing,
            'bg-[#D2D2D2]': examStatus === Status.Finished || examStatus === Status.Holiday || examStatus === Status.None || examStatus === Status.FutureExam
        }
    ]">
        <p class="text-[15px]"
            v-if="examStatus === Status.None || examStatus === Status.Holiday || examStatus === Status.Finished || examStatus === Status.FutureExam">
            {{
                examStatus === Status.None ? `Tidak ada ${$page.props.config.activity_type.toLowerCase()} saat ini.` :
                    examStatus === Status.Holiday ? `${$page.props.config.activity_type} libur saat ini.` :
                        examStatus === Status.Finished ? `${$page.props.config.activity_type} telah selesai.` :
                            `${$page.props.config.activity_type} akan dimulai pada ${examDate}`
            }}
        </p>
        <template v-else-if="examStatus === Status.StartingSoon || examStatus === Status.OnGoing">
            <p class="text-[15px] font-bold">{{ examStatus === Status.StartingSoon ? `${$page.props.config.activity_type} akan dimulai pada:` : `${$page.props.config.activity_type} akan berakhir pada: ` }}</p>
            <p class="text-[15px]">{{ examDate }}</p>
            <p class="text-[15px]">{{ examTimeRemaining }}</p>
        </template>
    </div>
</template>