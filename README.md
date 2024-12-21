# Выполнили Погребняк Степан Сиваселвамович и Кирилин Геннадий Дмитривевич

# Цель работы

![image](https://github.com/user-attachments/assets/619ee40b-8b4a-4ad2-98cb-39a4f954663c)




```
#include <stdint.h>
#include <stm32f4xx.h> // Убедитесь, что используется подходящая библиотека для вашего микроконтроллера

// Настройка GPIO для ШИМ
void initPWM(void) {
    // Включение тактирования GPIOA и TIM2
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;

    // Настройка PA5 в режим альтернативной функции (TIM2_CH1)
    GPIOA->MODER &= ~(3 << (5 * 2)); // Очистка битов
    GPIOA->MODER |= (2 << (5 * 2));  // Установка режима AF
    GPIOA->AFR[0] |= (1 << (5 * 4)); // Альтернативная функция TIM2_CH1

    // Настройка таймера TIM2 для ШИМ
    TIM2->PSC = 84 - 1;      // Предделитель (1 МГц)
    TIM2->ARR = 1000 - 1;    // Частота ШИМ = 1 кГц
    TIM2->CCR1 = 0;          // ШИМ с начальным коэффициентом заполнения = 0%
    TIM2->CCMR1 |= (6 << 4); // PWM mode 1
    TIM2->CCER |= TIM_CCER_CC1E; // Включение канала
    TIM2->CR1 |= TIM_CR1_CEN;    // Включение таймера
}

// Настройка АЦП
void initADC(void) {
    // Включение тактирования GPIOA и ADC1
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOAEN;
    RCC->APB2ENR |= RCC_APB2ENR_ADC1EN;

    // Настройка PA0 в режим аналогового входа
    GPIOA->MODER |= (3 << (0 * 2)); // Аналоговый режим

    // Настройка ADC1
    ADC1->CR2 |= ADC_CR2_ADON;      // Включение АЦП
    ADC1->SQR3 = 0;                 // Первый канал
    ADC1->SMPR2 |= ADC_SMPR2_SMP0; // Выбор времени выборки
}

// Чтение значения с АЦП
uint16_t readADC(void) {
    ADC1->CR2 |= ADC_CR2_SWSTART; // Запуск преобразования
    while (!(ADC1->SR & ADC_SR_EOC)); // Ожидание завершения
    return ADC1->DR; // Возврат значения
}

void delay(volatile uint32_t count) {
    while (count--);
}

int main(void) {
    initPWM(); // Инициализация ШИМ
    initADC(); // Инициализация АЦП

    uint16_t adcValue;
    while (1) {
        adcValue = readADC(); // Чтение с АЦП
        TIM2->CCR1 = adcValue / 16; // Регулировка ШИМ (яркость светодиода)
        delay(100000); // Задержка
    }
}

```
