# Accessibility Testing
Тестирование доступности (E2E). Проверяют, что наши страницы сайта соответствуют критериям доступности. Имеются вручную написанные тесты с помощью Playwright, которые проверяют:

Правильный путь навигации с помощью Tab'ов
Наличие фокуса на элементе
Наличие aria-label в элементах без текста (например крестик закрытия попапа)
Что проверяют?
Проверяют сайт по критериям доступности.

Когда пишутся Playwright тесты на доступность?
Пишутся после того как сверстан блок и в блоке присутствуют интерактивные элементы (кнопки, ссылки). Примеры тут

Также имеются тесты, внутри которых используется инструмент axe-core, который анализирует страницы сайта на соответствие критериям доступности.

Когда пишутся тесты c axe-core?
Пишутся после того как сверстана страница. Примеры тут


# Тестирование доступности

Тестировать нужно все, что касается взаимодействия пользователя с системой.

## Что именно тестировать?

- Атрибуты (role, tabindex, aria-атрибуты)
- Взаимодействие (клики, навигация с клавиатуры)
- Визуальная составляющая (тексты, контраст, расположение элементов и т.д.)

## Анализаторы доступности:

- [Eslint-plugin-jsx-a11y](https://www.npmjs.com/package/eslint-plugin-jsx-a11y) - линтер доступности JSX элементов
- [Stylelint-a11y](https://www.npmjs.com/package/stylelint-a11y) - линтер CSS
- [AccessLint](https://accesslint.com/) - апп для гитхаба, который находит проблемы с доступностью в пулреквестах в HTML
- [axe-core](https://www.npmjs.com/package/cypress-axe) - библиотека, интегрируемая в cypress 

![image info](./images/cypress-example.png)

*Пример работы axe в cypress*

![image info](./images/cypress-console.png)

*Подробная информация в консоли*

![image info](./images/axe-logs.png)

*[Улучшенные логи axe](https://github.com/denis-biruk/a11y-test-tabs/blob/master/cypress/support/commands/checkPageA11y.js)*


В cypress из коробки по прежнему нет поддержки браузерных событий (например tab click). Но есть плагин [Cypress-real-events](https://github.com/dmtrKovalenko/cypress-real-events) (MIT), который позволяет проверять браузерные события.
Ссылка на [stackoverflow](https://stackoverflow.com/questions/55009332/cypress-type-tab-key).

![image info](./images/cypress-real-events.png)

Playwright позволяет тестировать браузерные события, для него также есть плагин axe-core.

### Плюсы Playwright

- Тестирование нативных браузерных событий
- Есть плагин axe-core

### Минусы Playwright
- Интерфейс менее наглядный, чем в Cypress
- Для нас выше порог входа
- Синтаксис более многословный (async, переходы по url)


![image info](./images/alt-check-cypress.png)<br/>
*Пример проверки alt на Cypress*<br/>

![image info](./images/alt-check-playwright.png)<br/>
*Примеры проверки alt на Playwright*<br/>

![image info](./images/alt-check-cypress.png)<br/>
*Пример проверки alt на Cypress*<br/>

![image info](./images/alt-check-playwright.png)<br/>
*Примеры проверки alt на Playwright*<br/>

![image info](./images/title-check-cypress.png)<br/>
*Пример проверки title на Cypress*<br/>

![image info](./images/title-check-playwright.png)<br/>
*Примеры проверки title на Playwright*<br/>

![image info](./images/label-check-cypress.png)<br/>
*Пример проверки label на Cypress*<br/>

![image info](./images/label-check-playwright.png)<br/>
*Примеры проверки label на Playwright*<br/>

![image info](./images/placeholder-check-cypress.png)<br/>
*Пример проверки placeholder на Cypress*<br/>

![image info](./images/placeholder-check-playwright.png)<br/>
*Примеры проверки placeholder на Playwright*<br/>

![image info](./images/aria-check-cypress.png)<br/>
*Пример проверки aria на Cypress*<br/>

![image info](./images/aria-check-playwright.png)<br/>
*Примеры проверки aria на Playwright*<br/>

Выборка производится по aria ролям, определённых в спецификации W3C (пункт 5.2.8.4) - https://www.w3.org/TR/wai-aria-1.2/#roles


![image info](./images/tab-check-cypress.png)<br/>
*Пример проверки tab на Cypress*<br/>

![image info](./images/tab-check-playwright.png)<br/>
*Примеры проверки tab на Playwright*<br/>

![image info](./images/addressing-elements-playwright.png)<br/> 
*Пример обращения к элементам различными способами на Playwright*<br/>

Страница [Locators API](https://playwright.dev/docs/locators) в документации Playwright.

![image info](./images/contrast-ratio-check-cypress.png)<br/> 
*Пример проверки контрастности на Cypress с использованием пакета axe-core*<br/> 

![image info](./images/contrast-ratio-check-playwright.png)<br/> 
*Пример проверки контрастности на Playwright с использованием пакета axe-core*<br/> 

## Что нельзя протестировать автоматически?
- Цветовое восприятие<br/>
![image info](./images/color-perception.png)
- Избыточность лейблов<br/>
![image info](./images/redundant-labels.png)
- Визуальный порядок контента (tab order должен соответствовать реальному)
- Наложение интерфейса (например при зуме)<br/>
![image info](./images/ui-overlay.png) 
- Неправильная семантика (например email на таблицах)<br/>
![image info](./images/wrong-sematics.png)<br/> 
*Убираем семантику*

- Удобство использования
- Ловушки для фокуса (датапикеры, модалки)<br/>
![image info](./images/focus-traps.png)
- Контент: тексты, валидация форм, переводы<br/>
![image info](./images/form-validation.png)
![image info](./images/wrong-translation.png)
- Адекватность альтов<br/>
![image info](./images/wrong-alt.png)


[Доклад: Как протестировать accessibility руками](https://www.youtube.com/watch?v=-N34pdUcJf0&list=PLNSmyatBJig6ciyZ8A8zU0ZCngZxj858X&index=14)
[Доклад: npm run a11y-test](https://holyjs.ru/archive/2021%20Piter/talks/6zmpjnochngcpe7qksvglj/?referer=/archive/2021%20Piter/talks/)

### Инструменты 
- Автоматический отчет о доступности
- [W3C Валидатор](https://validator.w3.org/)
- Google Lighthouse
- [WAVE](https://wave.webaim.org/) Web Accessibility Evaluation Tool 
- [axeDevTools](https://chromewebstore.google.com/detail/axe-devtools-web-accessib/lhdoppojpmngadmnindnejefpokejbdd?utm_source=deque.com&utm_medium=referral&utm_campaign=axe_chrome_logo)

- Скринридеры (NVDA для Windows, VoiceOver для Mac)
