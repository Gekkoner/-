# FIXES_DETAILED.md - Конкретные исправления для кода

## БАГ #1: Удалить логику самопродажи

### Что искать в коде:
Найдите в JavaScript разделе функции, связанные с покупкой. Ищите:

```javascript
// УДАЛИТЬ ЭТИ СТРОКИ:
if (buyerId === sellerId) { ... }
if (currentUserId === seller.id) { ... }
selfDeliveryFlow()
isSellingToMyself
```

### Замена:
```javascript
// ❌ БЫЛО - УДАЛИТЬ:
if (buyerId === sellerId) {
  handleSelfDelivery(); // Функция самопродажи
  return;
}

// ✅ СТАЛО - ДОБАВИТЬ:
if (buyerId === sellerId) {
  alert("Вы не можете купить свой товар!");
  return; // Не продолжаем транзакцию
}
```

---

## БАГ #2: Согласовать характеры с именами

### Текущая проблема:
Дениса зовут с характером "бабушка" (или наоборот). Нужно привязать имена к характерам.

### Что искать:
```javascript
const personalities = ['бабушка', 'молодежь', 'бизнесмен', ...]
const names = ['Дениса', 'Ирина', 'Петя', ...]
function generateSeller() { ... }
```

### Решение - создать связь:
```javascript
// Заменить на:
const characterProfiles = {
  'babushka': {
    names: ['Людмила', 'Галина', 'Валентина', 'Евдокия', 'Тамара'],
    personality: 'бабушка',
    traits: ['медленная', 'осторожная', 'вежливая']
  },
  'molodezh': {
    names: ['Даша', 'Маша', 'Паша', 'Саша', 'Лена', 'Коля'],
    personality: 'молодежь',
    traits: ['быстрая', 'дерзкая', 'раскованная']
  },
  'businessman': {
    names: ['Сергей', 'Алексей', 'Иван', 'Борис', 'Владимир'],
    personality: 'бизнесмен',
    traits: ['профессиональная', 'серьезная', 'деловая']
  }
};

function generateSeller() {
  let profileType = Math.random();
  
  // Убрать "бабушку" для дорогих товаров
  if (itemPrice > 100000 && profileType < 0.33) {
    profileType = 0.5; // Выбрать молодежь вместо бабушки
  }
  
  const profile = profileType < 0.33 ? characterProfiles['babushka'] :
                  profileType < 0.66 ? characterProfiles['molodezh'] :
                                       characterProfiles['businessman'];
  
  const name = profile.names[Math.floor(Math.random() * profile.names.length)];
  return {
    name: name,
    personality: profile.personality,
    traits: profile.traits
  };
}
```

---

## БАГ #3: Переписать фразы при продаже

### Что искать:
```javascript
"Спасибо беру"
"Благодарю"
"Спасибо, беру!"
```

### Удалить:
Все фразы вида `"Спасибо беру"` - это фразы ПОКУПАТЕЛЯ, а не продавца.

### Заменить на фразы ПРОДАВЦА (когда ты продаешь):

```javascript
// Фразы продавца при принятии заказа:
const sellerAcceptancePhrases = [
  "Хорошо, оформляю доставку!",
  "Отлично, беру твой заказ!",
  "Ладно, паквую товар...",
  "Согласен, отправляю!"
];

// Фразы продавца при отправке доставки:
const sellerDeliveryPhrases = [
  "Доставка оформлена (Экспресс), 1 дн. в пути. Трек-номер: {TRACK}",
  "Отправил. Код доставки: {TRACK}. Приходит через день",
  "Доставка 3 дня. Трек: {TRACK}",
  "Посылка отправлена, номер {TRACK}. Следи по отслеживанию"
];

// Когда ТЫ продаешь:
function onSellItem(item, buyerInfo) {
  // 1. Отправить фразу принятия заказа ОТ СЕБЯ (продавца)
  const acceptPhrase = sellerAcceptancePhrases[
    Math.floor(Math.random() * sellerAcceptancePhrases.length)
  ];
  addMessage({
    from: 'me', // ОТ МЕНЯ (продавца)
    text: acceptPhrase,
    timestamp: new Date()
  });

  // 2. Генерировать трек-номер
  const trackNumber = generateTrackNumber();
  
  // 3. Отправить информацию о доставке ОТ СЕБЯ
  const deliveryMsg = sellerDeliveryPhrases[
    Math.floor(Math.random() * sellerDeliveryPhrases.length)
  ].replace('{TRACK}', trackNumber);
  
  addMessage({
    from: 'me', // ОТ МЕНЯ (продавца)
    text: deliveryMsg,
    timestamp: new Date()
  });
}

// Функция генерации трек-номера:
function generateTrackNumber() {
  const letters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  const numbers = '0123456789';
  
  let track = '';
  // 2 буквы
  track += letters[Math.floor(Math.random() * 26)];
  track += letters[Math.floor(Math.random() * 26)];
  
  // 7 цифр
  for (let i = 0; i < 7; i++) {
    track += numbers[Math.floor(Math.random() * 10)];
  }
  
  return track; // Пример: A8ZY879265
}
```

---

## Итоговый чек-лист применения:

- [ ] **Баг #1**: Найти и удалить `if (buyerId === sellerId)`
- [ ] **Баг #1**: Удалить функции самопродажи
- [ ] **Баг #2**: Заменить `personalities` и `names` на `characterProfiles`
- [ ] **Баг #2**: Добавить проверку на цену товара (убрать "бабушку" для дорогих)
- [ ] **Баг #3**: Найти и удалить все `"Спасибо беру"`
- [ ] **Баг #3**: Добавить `sellerAcceptancePhrases`
- [ ] **Баг #3**: Добавить `sellerDeliveryPhrases`
- [ ] **Баг #3**: Добавить функцию `generateTrackNumber()`
- [ ] **Баг #3**: Изменить логику отправки сообщений так, чтобы `from: 'me'` когда ты продаешь

---

## Как применить:

1. Откройте `index-10.html` в текстовом редакторе
2. Найдите секцию `<script>` (где JavaScript код)
3. Примените изменения следуя пунктам выше
4. Сохраните файл
5. Откройте в браузере и тестируйте

**Все фразы, коды и функции написаны выше - просто скопируйте в свой код!**
