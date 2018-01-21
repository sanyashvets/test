<template>
  <div class="flex-center">
    <div class="list-wrap flex-center" v-clickaway="close">
      <div>
        <label for="input">Поиск:</label>
        <input type="text" id="input" v-model="value" @input="handleInput" @keydown.down="pressKey"
               @keydown.up="pressKey">
      </div>
      <p>Список мега дизайна будет отображаться ниже</p>
      <ul ref="ul" tabindex="-1" v-show="showList">
        <li v-for="(i, index) in result"
            :key="i.id"
            @keydown.down="pressKey(1, index)"
            @keydown.up="pressKey(-1, index)"
            @keydown.enter="pressEnter(i.name)"
            :ref="'item-' + index" :tabindex="index">
          <span>{{index + 1}}.</span>
          <span v-html="highlight(i.name, value)"></span>
        </li>
        <li v-show="!result.length">No match found</li>
      </ul>
    </div>

    <div class="tip">
      <ul>
        <li>
          <p>Имена товаров генерируются в формате <code>`Some product #${номер (в моем случае - индекс итерации)}`</code></p>
        </li>
        <li>
          <p>Я не делал никаких дизайнов товаров и всего прочего, т.к. посчитал, что для проверки это вообще не нужно.
            Список появляется/скрывается, подсветка/задержка на инпут/навигация и выбор по клавиатуре есть и больше ничего не нужно </p>
          <p>В случае, если бы это был компонент - данные приходили бы через <code>props</code> и там был бы еще дропдаун для селекта и всё такое.. не хотел тратить на это время</p>
          <p>RxJS не юзал ввиду отстутсвия необходимости и обязательности условий его использования</p>
        </li>
      </ul>
    </div>
    <p></p>
  </div>
</template>

<script>

  /**
   * Отложенный вызов функции
   * https://learn.javascript.ru/task/debounce
   *
   * @param f {Function} - функция
   * @param ms {Number} - мс, через который будет вызвана функция f
   * @return {Function}
   */
  function debounce(f, ms) {

    let timer = null

    return function (...args) {
      const onComplete = () => {
        f.apply(this, args)
        timer = null
      }

      if (timer) {
        clearTimeout(timer)
      }

      timer = setTimeout(onComplete, ms)
    }
  }

  export default {
    name: 'HelloWorld',
    data() {
      return {
        value: '', // значение инпута с поиском
        recordsCount: 30, // кол-во записей для генерации
        data: [], // осн. массив с данными
        searchResult: [], // массив с результатами поиска
        showList: false, // флаг видимости списка результатов поиска
      }
    },
    methods: {
      /**
       * Ловим событие ввода в инпут
       *
       * запускает поиск и открывает/скрывает список результатов
       */
      handleInput: debounce(function () {
        this.showList = !!this.value
        this.filter(this.value)
      }, 500),
      /**
       * Фильтрация по совпадению значений с инпутом
       *
       * @param str {String} - значение инпута
       */
      filter(str) {
        if (!str) {
          this.searchResult = []
          return false
        }
        this.searchResult = this.data.filter(i => i.name.toLowerCase().includes(str.toLowerCase()))
      },
      /**
       * Отлавливаем нажатие клавиши Enter
       *
       * Заполняет текущее значение поиска выбранным и скрывает список
       *
       * @param name {String} имя выбранного товара
       */
      pressEnter(name) {
        this.value = name
        this.showList = false
      },
      /**
       * Фокус элемента по навигации с клавиатуры
       *
       * @param direction {Number} - Направление срелки. 1 - вниз, -1 - вверх
       * @param index {Number} - индекс
       *
       */
      pressKey(direction = 1, index) {
        const resultLen = this.result.length
        if (!resultLen) return
        let param = 'item-'
        if (index === undefined || resultLen === 1) {
          param += 0
        } else {
          let possibleIndex
          if (index === resultLen - 1 && direction === 1) {
            possibleIndex = 0
          } else if (index === 0 && direction === -1) {
            possibleIndex = resultLen - 1
          } else {
            possibleIndex = index + direction
          }
          param += possibleIndex
        }
        this.$refs[param][0].focus()
      },
      /**
       * https://stackoverflow.com/a/280805/1638298
       * used in this.highlight
       * @param str
       * @returns {String}
       * @private
       */
      _preg_quote(str) {
        return (str + '').replace(/([\\\.\+\*\?\[\^\]\$\(\)\{\}\=\!\<\>\|\:])/g, '\\$1')
      },
      /**
       * Получить хтмл код с значением поиска
       *
       * https://stackoverflow.com/a/280805/1638298
       * @param data
       * @param search
       *
       * @return {String} html-код с выделенным значением
       */
      highlight(data, search) {
        if (!data || !search) return data
        return data.replace(new RegExp('(' + this._preg_quote(search) + ')', 'gi'), '<span class=\'highlighted\'>$1</span>')
      },
      /**
       * Спрятать список
       *
       */
      close() {
        this.showList = false
      },
      /**
       * Заполняет список "товаров" какими-то значениями
       *
       * Было лениво делать это вручную
       */
      fillDataValues() {
        for (let i = 0; i < this.recordsCount; i++) {
          this.data.push({
            id: i,
            name: `Some product #${i + 1}`,
            description: 'some description',
            photo: 'https://somesite.com/someimg.png',
          })
        }
      },
    },
    created() {
      this.fillDataValues()
    },
    computed: {
      /**
       * получаем 5 совпадений из всех
       *
       * @return {Array}
       */
      result() {
        return this.searchResult.slice(0, 5)
      },
    },
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style>

  .flex-center {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }

  .list-wrap {
    width: 400px;
    border: 1px solid rgba(0, 0, 0, .3);
    padding: 30px;
  }

  .highlighted {
    color: green;
    font-weight: bold;
  }

  ul {
    list-style-type: none;
    padding: 0;
    margin: 10px 0 0;
  }

  ul li:focus {
    background: orange;
  }

</style>
