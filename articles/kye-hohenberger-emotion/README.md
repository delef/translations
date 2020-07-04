![](https://cdn-images-1.medium.com/max/800/1*gGzRmUKNOC_X7klFjTk8EA.png)<br/>
*Иллюстрация от [https://twitter.com/ChrisMCarrasco](https://twitter.com/ChrisMCarrasco)*

# emotion. Следующее поколение CSS-in-JS

*Перевод статьи [Kye Hohenberger](https://t.co/SY7htPqRHF): [emotion. The Next Generation of CSS-in-JS](https://medium.com/@tkh44/emotion-ad1c45c6d28b).*

[Emotion](https://github.com/tkh44/emotion) - это [высокопроизводительная](https://github.com/tkh44/emotion/blob/master/docs/benchmarks.md), легкая библиотека css-in-js. Основная идея исходит из библиотеки [glam](https://github.com/threepointone/glam) [Сунила Пая](https://medium.com/@threepointone), и её философия изложена здесь. Ключевая идея очень проста. Вам не нужно жертвовать производительностью ради удобства разработчиков при написании CSS. Emotion минимизируют стоимость исполнения css-in-js, анализируя ваши стили с помощью babel и PostCSS. Ядро библиотеки в рантайме занимает 2,3 кб и поддержка React — 4 кб. Всего.

## Первый взгляд

API покажется вам знакомым, если вы знаете [styled-components](https://github.com/styled-components/styled-components). Мы также черпали вдохновение в [glamorous](https://github.com/paypal/glamorous) и [css-modules](https://github.com/css-modules/css-modules#composition).

```javascript
const imageBase = css`
  width: 32px;
  height: 32px;
  border-radius: 50%;
`
const Avatar = styled.img`
  composes: ${imageBase};
  border: 2px solid ${p => p.theme.borderColor}

  @media(min-width: 420px) {
    width: 96px;
    height: 96px;
  }
`
```

## Бенчмарки

![](https://cdn-images-1.medium.com/max/800/1*6SuuwL3uemPu-0s4-zfPcQ.png)

Этот [бенчмарк](https://github.com/tkh44/CSS-IN-JS-Benchmarks) нагружает библиотеки, очень быстро изменяя динамическое значение в блоке стиля.

Против styled-components - **emotion более чем в 25 раз быстрее при использовании высокодинамичных значений props и при этом меньше на треть**.

*Styled-components предупреждают об опасности использовании этого паттерна и вместо этого рекомендуют использовать инлайн-стили для высокодинамических значений. Emotion не имеют таких ограничений.*

Против glamorous - **emotion до 2,5 раз быстрее на ререндере и в половину меньше**.

## CSS

`import { css } from 'emotion'`

CSS это ❤️ emotion. Большая часть API, включая `styled`, является просто обёрткой над `css`.

```javascript
const imageBase = css`
  width: 32px;
  height: 32px;
  border-radius: 50%;
`
```

`css` - это [тегированный литерал шаблона](https://developers.google.com/web/updates/2015/01/ES6-Template-Strings#tagged_templates), который принимает стандартный css-текст с несколькими дополнительными функциями, такими как вложенность, псевдоселекторы и медиа-запросы.

`css` возвращает строковое имя класса, которое может использоваться для любого элемента. Для нашего блока стиля `imageBase`, описанного выше, это будет что-то вроде `css-imageBase-12345`.

Чтобы помочь с композицией, `css` принимает специальное свойство в блоке стиля, называемое `composes`.

С помощью `composes` мы можем использовать наш `imageBase`, чтобы быстро создавать стили аватаров.

```javascript
const avatarStyle = css`
  composes: ${imageBase};
  border: 1px solid #7519E5
`
```

Внутренняя реализация emotion добавит эти css-классы к сгенерированному результату. `avatarStyle` получит имя класса вида `css-imageBase-12345 css-avatarStyle-12345`. Это позволяет создать действительно мощную композицию, которая особенно хорошо показывает себя вместе со `styled`.

```javascript
const imageStyles = css({
  width: 96,
  height: 96
})
```

*Объекты стилей не обрабатываются авто-префиксером.*

## styled

`import styled from 'emotion/react'`

`styled` это тонкая обертка вокруг `css`, которая поддерживает тот же стиль текста и выражений, что и `css`.

Использование `styled` очень похоже на использование styled-components.

```javascript
const Avatar = styled.img`
  width: 32px;
  height: 32px;
  border-radius: 50%;
`
```

`styled` также работает как вызов функции. Первым аргументом может быть любой html-тег или React-компонент, который имеет поддержку свойства className.

```javascript
const BigAvatar = styled(Avatar)`
  width: 96px;
  height: 96px;
`
```

Вы также можете использовать другой стилизованный компонент в качестве селектора.

```javascript
const Heading = styled.img`
  font-family: serif;
`
const Header = styled.header`
  display: flex;
  ${Heading} {
    font-size: 48px;
    align-self: flex-end;
  }
`
```

Композиция также работает.

```javascript
const imageBase = css`
  width: 32px;
  height: 32px;
  border-radius: 50%;
`
const Avatar = styled.img`
  composes: ${imageBase};

  @media(min-width: 420px) {
    width: 96px;
    height: 96px;
  }
`
```

В `styled`, значение интерполяции `composes` может быть функцией. Она вызывается с текущими props во время рендера. Эта функция должна возвращать className или объект стиля.

```javascript
const types = {
  success: css`
    background: green;
  `,
  error: css`
    background: red;
  `,
}
const Alert = styled.div`
  composes: ${props => types[props.type]};
  padding: 10px;
`
```

*Помните, что css просто возвращает className, так что это работает.*

Истинная сила композиции раскрывается при её использовании с чем-то вроде [styled-system](https://medium.com/@tkh44/emotion-ad1c45c6d28b) Брента Джексона.

[https://twitter.com/tkh44/status/883836708453855232/photo/1](https://twitter.com/tkh44/status/883836708453855232/photo/1)

## Поддержка тем

`import { ThemeProvider } from 'emotion/react'`

Поддержка тем реализована с помощью библиотеки [theming](http://npmjs.com/package/theming). Детали API подробно изложены [по ссылке](https://github.com/iamstarkov/theming/blob/master/README.md#api). Она основана на системе тем styled-components и мощно протестирована, что сделало её беспроблемной.

Всякий раз, когда вы предоставляете тему `ThemeProvider`, любой стилизованный компонент имеет доступ к этим стилям через `props.theme`. Неважно, как глубоко вложен ваш компонент внутри `ThemeProvider`, у вас все ещё есть доступ к `props.theme`.

```javascript
const theme = {
  white: '#f8f9fa',
  purple: '#8c81d8',
  gold: '#ffd43b'
}
const Avatar = styled.img`
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: 1px solid ${props => props.theme.gold}
`
<ThemeProvider theme={theme}>
  <Avatar src={avatarUrl}>
    hello world
  </Avatar>
</ThemeProvider>
```

---

## Extract Mode против Inline Mode

Babel-плагин для emotion по умолчанию работает в так называемом «extract mode». В этом режиме мы берём все ваши css, определенные в каждом файле, извлекаем их в `[filename].emotion.css` и автоматически импортируем в верхнюю часть вашего JavaScript-файла. Динамические значения обрабатываются с помощью CSS-переменных. Единственные обновления во время выполнения - это просто изменения в CSS-переменных! Недостатки режима «extract mode»  — [отсутствие поддержки IE11 из-за CSS-переменных](http://caniuse.com/#feat=css-variables) и невозможность извлечь критический CSS для рендеринга на стороне сервера.

В «inline mode» emotion работает несколько иначе.

В этом режиме используется [CSS Object Model (CSSOM)](http://blog.frankmtaylor.com/2013/10/15/exploring-the-cssom-making-a-css-object-analyzer/) для управления CSS из JavaScript. Так же работает любая другая библиотека css-in-js. Что отличает emotion, так это то, что мы можем прекомпилировать все ваши CSS-правила. **Нам не нужно анализировать и дополнять ваши стили префиксами в рантайме, потому что мы уменьшаем ваши стили до непосредственных вызовов [insertRule](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/insertRule).**

Самые большие преимущества режима «inline mode» заключаются в том, что он работает на IE9 и выше, и у него есть поддержка рендеринга на стороне сервера с помощью `extractCritical`. Вот отличный пример этого в репозитории [next.js](https://github.com/zeit/next.js/tree/v3-beta/examples/with-emotion).

---

## Под капотом

*Следующий пример предназначен для режима «inline mode». Режим «extract mode» работает практически также.*

Этот код

```javascript
const H1 = styled.h1`
  font-size: 48px;
  color: ${props => props.color};
`
```

С помощью Babel компилируется в следующий

```javascript
const H1 = styled(
  'h1',
  ['css-H1-duiy4a'], // generated class names
  [props => props.color], // dynamic values
  function createEmotionStyledRules (x0) {
    return [`.css-H1-duiy4a { font-size:48px; color:${x0} }`]
  }
)
```

Всякий раз, когда вызывается метод рендеринга `H1`:

1. `styled` проходит через каждое динамическое значение в вашем блоке CSS. Когда он итерируется через этот список, любое значение, которое является функцией, вызывается и получает текущие `props`.
2. `createEmotionStyledRules` вызывается с использованием конечных значений с шага 1 в качестве аргументов. В нашем примере результатом `props => props.color` будет `x0`.
3. Результат этого вызова затем вставляется в обёртку [StyleSheet](https://developer.mozilla.org/en-US/docs/Web/API/StyleSheet) emotions, где он агрессивно кэшируется.
4. Сгенерированные имена классов добавляются к свойству `className` компонента.

Взгляните на исходники [`styled`](https://github.com/tkh44/emotion/blob/master/src/react.js) и [`css`](https://github.com/tkh44/emotion/blob/master/src/index.js#L35).

## Бонус

### CSS-анимации

Анимации поддерживаются с помощью функции [keyframes](https://github.com/tkh44/emotion/blob/master/docs/keyframes.md).

### Свойство css

Любой компонент в вашем приложении, который принимает свойство `className`, теперь может принимать свойство `css`.

```javascript
const flexCenter = css`
  display: flex;
  align-items: center;
  justify-content: center;`

<div
  css={`
    composes: ${flexCenter};
    width: 128px;
    height: 128px;
    background-color: #8c81d8;
    border-radius: 4px;

    img {
      width: 96px;
      height: 96px;
      border-radius: 50%;
      transition: all 400ms ease-in-out;

      &:hover {
        transform: scale(1.2);
      }
    }
  `}
>
```

Свойство себя точно так же, как функция `css`. Полученное имя класса добавляется к свойству `className` элемента.

---

**Пойдите, напишите немного CSS.**

Я хочу поблагодарить [Сунила Пая](https://medium.com/@threepointone) за его руководство, терпение и за доверие ко мне, в том, чтобы выполнить его идею. 🙏

Огромное спасибо [Митчеллу Гамильтону](https://medium.com/@hamiltown), потому что без него emotion не работали бы так хорошо. Его [вклад](https://github.com/tkh44/emotion/commits?author=mitchellhamilton) был неоценим в том, чтобы мы достигли того, чего достигли.

Спасибо всем [контрибьюторам](https://github.com/tkh44/emotion/graphs/contributors), создателям ишью и тестировщикам ранних релизов!

[emotion.sh](https://emotion.sh/) — вебсайт

[slack.emotion.sh](http://slack.emotion.sh/) — канал в Slack, заходите поздороваться! 👋

Спасибо, [Nadim](https://medium.com/@shinzui).

---

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/девшахта/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*

[Статья на Medium](https://medium.com/devschacht/emotion-ebe5fe3ba116)
