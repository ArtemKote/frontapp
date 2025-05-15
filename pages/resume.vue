<template>
    <div class="max-w-5xl mx-auto p-6 bg-gray-900 text-white rounded-lg shadow-md">
      <!-- Иконки в шапке -->
      <div class="flex justify-start gap-2 mb-4">
        <button v-for="icon in icons" :key="icon" class="bg-gray-700 p-2 rounded hover:bg-gray-600">
          <UIcon :name="icon" />
        </button>
      </div>
  
      <!-- Основной блок -->
      <div class="flex flex-wrap gap-6">
        <!-- Аватар -->
        <UAvatar 
          :src="data?.photo || '/avatar_thumb.png'" 
          size="xl" 
          class="w-32 h-32 rounded" 
        />
  
        <!-- Информация -->
        <div class="flex-1">
          <h1 class="text-2xl font-bold">{{ data?.name }}</h1>
          <p class="text-sm text-blue-400">
            Кандидат на вакансию: <span class="underline text-blue-300">{{ data?.listing_id }}</span>
            <span class="text-gray-400 ml-2">({{ new Date(data?.date || '').toLocaleDateString('ru-RU') }} Отклик)</span>
          </p>
  
          <div class="mt-4 flex flex-col gap-2 text-sm">
            <div class="flex items-center gap-2">
              <UIcon name="i-heroicons-phone" class="text-yellow-500" />
              <a :href="'tel:' + data?.phone" class="hover:text-yellow-400">{{ data?.phone }}</a>
            </div>
            <div class="flex items-center gap-2">
              <UIcon name="i-heroicons-envelope" class="text-green-500" />
              <a :href="'mailto:' + data?.email" class="hover:text-green-400">{{ data?.email }}</a>
            </div>
            <div class="flex items-center gap-2">
              <UIcon name="i-heroicons-map-pin" class="text-blue-500" />
              <span>{{ data?.town }}</span>
            </div>
            <!-- Соц. сети -->
            <div class="flex gap-2 mt-2">
              <UIcon name="i-simple-icons-whatsapp" class="text-green-500" />
              <UIcon name="i-simple-icons-viber" class="text-purple-500" />
              <UIcon name="i-simple-icons-telegram" class="text-blue-500" />
            </div>
          </div>
        </div>
      </div>
  
      <!-- Кнопки действий -->
      <div class="flex flex-wrap gap-2 mt-6">
        <UButton 
          :color="data?.status === 'viewed' ? 'primary' : 'neutral'"
          class="px-4 py-2 rounded text-sm"
        >
          {{ data?.status === 'viewed' ? 'Просмотрено' : 'Новое' }}
        </UButton>
        <UButton color="neutral" class="px-4 py-2 rounded text-sm">Создать видеозвонок</UButton>
        <UButton color="neutral" class="px-4 py-2 rounded text-sm">Запланировать событие</UButton>
        <UButton color="primary" class="px-4 py-2 rounded text-sm">Отправить запрос</UButton>
      </div>
  
      <!-- Статус -->
      <div class="mt-8">
        <h2 class="font-bold mb-2 text-sm">Статус рассмотрения:</h2>
        <div class="flex flex-wrap gap-2 text-xs">
          <UBadge 
            v-for="(status, index) in steps" 
            :key="status.value"
            :color="getStepColor(status.value)"
            variant="soft"
            class="cursor-default"
          >
            {{ status.label }}
          </UBadge>
        </div>
      </div>
  
      <!-- Прочая информация -->
      <div class="mt-6">
        <h2 class="font-bold mb-2 text-sm">Информация о кандидате</h2>
        <div class="space-y-2">
          <p class="text-xs text-gray-300">Дата рождения: {{ new Date(data?.birth_date || '').toLocaleDateString('ru-RU') }}</p>
          <p class="text-xs text-gray-300">Возраст: {{ data?.age }} лет</p>
          <p class="text-xs text-gray-300">Город: {{ data?.town }}</p>
          <p class="text-xs text-gray-300 mt-4 whitespace-pre-line">{{ data?.description }}</p>
        </div>
      </div>
  
      <!-- Портфолио -->
      <div class="bg-cyan-600 p-4 mt-6 rounded" v-if="data?.portfolios">
        <p class="text-white font-bold text-sm">Файлы портфолио:</p>
        <div class="mt-2 space-y-1">
          <template v-if="Array.isArray(data.portfolios)">
            <a 
              v-for="(file, index) in data.portfolios" 
              :key="index"
              :href="file.url" 
              class="block underline text-white text-sm hover:text-gray-200"
            >
              {{ file.name }}
            </a>
          </template>
        </div>
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
  const icons = [
    'i-heroicons-document',
    'i-heroicons-document-text',
    'i-heroicons-trash',
    'i-heroicons-heart'
  ];
  
  interface ResumeData {
    id: number;
    name: string;
    description: string;
    date: string;
    status: string;
    portfolios: null | Array<{ name: string; url: string }>;
    town: string;
    phone: string;
    age: number;
    birth_date: string;
    email: string;
    listing_id: number;
    photo: string;
  }
  
  // Получение данных с API
  const { data, pending, error } = await useFetch<ResumeData>('https://dev.jobcart.ru/wp-json/test/v2/app');
  
  // Определяем все возможные статусы и их порядок
  const steps = [
    { label: 'Новое', value: 'new' },
    { label: 'Просмотрено', value: 'viewed' },
    { label: 'Ожидание ответа соискателя', value: 'waiting_response' },
    { label: 'Принятие решения', value: 'decision' },
    { label: 'Отправлено приглашение', value: 'invited' },
    { label: 'Принят', value: 'accepted' },
    { label: 'Назначено собеседование', value: 'interview_scheduled' },
    { label: 'Проведено собеседование', value: 'interview_completed' },
    { label: 'Отклонено/Отказ', value: 'rejected' },
    { label: 'Не дошел', value: 'not_arrived' },
    { label: 'Архивировано', value: 'archived' }
  ];
  
  // Функция для определения цвета статуса
  const getStepColor = (stepValue: string) => {
    // Находим индекс текущего шага и проверяемого шага
    const currentStepIndex = steps.findIndex(s => s.value === data.value?.status);
    const stepIndex = steps.findIndex(s => s.value === stepValue);
    
    // Если статус не найден или это конечные статусы
    if (currentStepIndex === -1 || ['rejected', 'not_arrived', 'archived'].includes(stepValue)) {
      return 'neutral';
    }
    
    // Если шаг уже пройден
    if (stepIndex <= currentStepIndex) {
      return 'primary';
    }
    
    // Если шаг еще не достигнут
    return 'neutral';
  };
  </script>

  <style>
  .cursor-default {
    cursor: default;
  }
  </style>