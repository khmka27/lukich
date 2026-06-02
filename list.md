Скорее всего, у тебя сайт на телефоне “дёргается” из-за сочетания нескольких вещей в CSS:

1. **Секции используют сразу `100vh`, `100svh`, `100dvh`**

У тебя есть такое:

```css
min-height: 100vh;
min-height: 100svh;
min-height: 100dvh;
```

например у `.hero` и `.next-part`.

На телефонах высота экрана постоянно меняется, когда появляется/исчезает адресная строка браузера. Особенно `100dvh` — это **dynamic viewport height**, он пересчитывается во время скролла. Из-за этого секция может слегка менять высоту, и визуально кажется, что сайт дёргается.

Лучше оставить так:

```css
.hero,
.next-part {
  min-height: 100svh;
}
```

или так:

```css
.hero,
.next-part {
  min-height: 100vh;
}
```

Я бы для твоего случая попробовал сначала:

```css
.hero,
.next-part {
  min-height: 100svh;
}
```

То есть заменить вот это:

```css
min-height: 100vh;
min-height: 100svh;
min-height: 100dvh;
```

на это:

```css
min-height: 100svh;
```

---

2. **Кнопка вниз постоянно анимируется**

У тебя есть:

```css
.scroll-down-btn--always-visible {
  animation: bounceDown 1.2s ease-in-out infinite;
}
```

И сама анимация:

```css
@keyframes bounceDown {
  0%,
  100% {
    transform: translateX(-50%) translateY(0);
  }
  50% {
    transform: translateX(-50%) translateY(8px);
  }
}
```

На телефоне постоянная анимация `transform` может давать микродёрганья, особенно вместе со скроллом.

Можно временно отключить на мобилке:

```css
@media (max-width: 920px) {
  .scroll-down-btn,
  .scroll-down-btn--always-visible {
    animation: none;
  }
}
```

---

3. **`background-glow` с `position: fixed`, `filter: blur()` и анимацией**

У тебя есть большие размазанные круги:

```css
.background-glow {
  position: fixed;
  filter: blur(65px);
}
```

и они ещё анимируются:

```css
animation: driftA 9s ease-in-out infinite alternate;
animation: driftB 12s ease-in-out infinite alternate;
```

На мобильных `filter: blur()` + `position: fixed` + анимация — частая причина лагов и дёрганий.

Для телефона лучше отключить анимацию или сами glow:

```css
@media (max-width: 920px) {
  .background-glow {
    animation: none;
    filter: blur(40px);
  }
}
```

Если всё равно дёргается, жёстче:

```css
@media (max-width: 920px) {
  .background-glow {
    display: none;
  }
}
```

---

4. **3D-модель тоже может нагружать телефон**

У тебя подключён `model-viewer` с `auto-rotate`, `camera-controls`, `autoplay`.

Но сейчас у `body` стоит класс:

```html
<body class="model-off"></body>
```

и в CSS:

```css
body.model-off .hero__model {
  display: none;
}
```

То есть модель у тебя сейчас скрыта.

Если ты уберёшь `model-off`, на телефоне сайт может начать дёргаться сильнее из-за 3D.

---

Я бы первым делом сделал вот такой фикс:

```css
.hero,
.next-part {
  min-height: 100svh;
}

@media (max-width: 920px) {
  .hero,
  .next-part {
    min-height: 100svh;
  }

  .background-glow {
    animation: none;
    filter: blur(40px);
  }

  .scroll-down-btn,
  .scroll-down-btn--always-visible {
    animation: none;
  }
}
```

И убрал из `.hero` и `.next-part` строки:

```css
min-height: 100vh;
min-height: 100dvh;
```

Главный подозреваемый именно `100dvh`, потому что на телефоне высота viewport меняется при скролле, и блоки могут пересчитываться прямо во время движения.
