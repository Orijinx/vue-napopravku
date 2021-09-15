<template>
  <v-app>
    <!-- NAV BAR -->
    <nav>
      <!-- HAMBURGER NAV FOR MOBILE -->
      <v-navigation-drawer
        absolute
        temporary
        v-model="drawer"
        class="hidden-md-and-up"
      >
        <v-list-item>
          <v-list-item-content>
            <v-list-item-title class="text-md-h6"> QUOTES </v-list-item-title>
            <v-list-item-subtitle> Меню </v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>

        <v-divider></v-divider>
        <v-spacer></v-spacer>
        <v-list nav>
          <v-list-item v-for="(item, i) in menuItems" :key="`menuitem${i}`">
            <v-list-item-icon>
              <v-icon>{{ item.icon }}</v-icon>
            </v-list-item-icon>
            <v-list-item-content>
              <v-btn @click="dialog = !dialog" text dense>
                {{ item.title }}</v-btn
              >
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-navigation-drawer>
      <v-app-bar app dark>
        <v-app-bar-nav-icon
          @click.stop="drawer = !drawer"
          class="hidden-md-and-up"
        ></v-app-bar-nav-icon>

        <v-text-field v-text="'Quotes'"></v-text-field>
        <v-spacer></v-spacer>
        <v-item-group class="hidden-sm-and-down">
          <v-item>
            <v-btn
              @click="dialog = !dialog"
              v-for="(item, i) in menuItems"
              text
              :key="`menuitem${i}`"
            >
              <v-icon left v-html="item.icon"></v-icon>
              {{ item.title }}</v-btn
            >
          </v-item>
        </v-item-group>
      </v-app-bar>
    </nav>
    <v-main>
      <v-item-group>
        <v-container>
          <!-- ALERT ERRORS -->
          <v-alert
            v-for="(alert, a) in errors"
            :key="`errors${a}`"
            border="bottom"
            color="red"
            outlined
            type="error"
          >
            <p v-if="alert.text">
              {{ alert.text }}
            </p>
            <p v-if="alert.tags">
              {{ alert.tags }}
            </p>
            <p v-if="alert.author">
              {{ alert.author }}
            </p>
          </v-alert>
          <!-- LOADER -->
          <v-row v-if="loading">
            <v-progress-linear
              indeterminate
              rounded
              color="cyan"
            ></v-progress-linear>
          </v-row>
          <!-- MAIN SECTION -->
          <v-row v-else>
            <v-col
              v-for="(item, i) in response.data"
              :key="`response.data${i}`"
              cols="12"
              md="6"
            >
            <!-- CARDS OF QUOTES -->
              <v-item>
                <v-card elevation="2">
                  <v-card-title class="text-lg-h6">{{
                    item.text
                  }}</v-card-title>
                  <v-card-subtitle>{{
                    BeautyDate(item.created_at)
                  }}</v-card-subtitle>
                  <!-- <v-card-subtitle >{{item.tags }}</v-card-subtitle> -->
                  <div class="d-flex flex-row sm-1">
                    <v-subheader>#</v-subheader>
                    <v-subheader
                      v-for="(tag, t) in item.tags"
                      :key="`item.tags${t}`"
                      >{{ tag.name }}</v-subheader
                    >
                  </div>

                  <v-card-subtitle>Автор: {{ item.author }}</v-card-subtitle>
                </v-card>
              </v-item>
            </v-col>
          </v-row>
          <v-row>
            <!-- CONTROL PANEL OF PAGES -->
            <v-toolbar dense flat>
              <v-toolbar-title left
                >{{ response.current_page }}
                <!-- /{{
                    response.last_page
                  }} -->
              </v-toolbar-title>
              <v-container>
                <v-toolbar-items>
                  <v-btn
                    @click="LoadPage(response.prev_page_url)"
                    text
                    v-if="response.prev_page_url"
                    >Назад</v-btn
                  >

                  <v-spacer></v-spacer>
                  <v-btn
                    @click="LoadPage(response.next_page_url)"
                    text
                    left
                    v-if="response.next_page_url"
                    >Вперед</v-btn
                  >
                </v-toolbar-items>
              </v-container>
            </v-toolbar>
          </v-row>
        </v-container>
      </v-item-group>
      <!-- ALERT TOAST -->
      <v-snackbar v-model="snackbar" :timeout="timeout">
        Сообщение: {{ msg }}

        <template v-slot:action="{ attrs }">
          <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
            Закрыть
          </v-btn>
        </template>
      </v-snackbar>
    </v-main>
    <!-- FOOTER -->
   <v-footer  dark padless>
    <v-col
      class="text-center"
      cols="12"
    >
      {{ new Date().getFullYear() }} — <strong>QUOTES</strong>
    </v-col>
  </v-footer>
    <!-- MODAL INPUT -->
    <v-dialog v-model="dialog" width="500">
      <v-form ref="form" v-model="valid">
        <v-card>
          <v-card-title>
            <span class="text-h5">Добавить цитату</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col>
                  <v-textarea
                    v-model="postData.text"
                    label="Текст"
                    :rules="RequiredRules"
                    required
                    hint="Напишите текст вашей цитату сюда."
                  ></v-textarea>
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12" sm="6" md="4">
                  <v-text-field
                    v-model="postData.author"
                    label="Автор"
                    :rules="RequiredRules"
                    required
                    hint="Ваше имя, псевдоним или прозвище."
                  ></v-text-field>
                </v-col>

                <v-col cols="12" sm="6">
                  <v-autocomplete
                    @click="GetTagList"
                    v-model="postData.tags"
                    :items="tags"
                    :rules="MaxItemRules"
                    counter="3"
                    open-on-clear="Загрузка тэгов..."
                    item-text="name"
                    item-value="id"
                    required
                    label="Тэги"
                    multiple
                    hint="Добавьте теги. Миниму один, максимум 3"
                  ></v-autocomplete>
                </v-col>
              </v-row>
            </v-container>
            <small>*Все поля обязательны к заполнению</small>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="dialog = false">
              Закрыть
            </v-btn>
            <v-btn
              color="blue darken-1"
              :disabled="!valid"
              text
              @click="PostQuote()"
            >
              Добавить
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-form>
    </v-dialog>
  </v-app>
</template>

<script>
export default {
  computed: {
    menuItems() { //Меню
      return [
        {
          icon: "mdi-pencil",
          title: "Добавить цитату",
        },
      ];
    },
  },

  methods: {
    BeautyDate(date) { //Вывод красивой даты 
      let d = new Date(date);
      let serialized =
        d.getDay() +
        "." +
        d.getMonth() +
        "." +
        d.getFullYear() +
        " " +
        d.getHours() +
        ":" +
        d.getMinutes();
      return serialized;
    },
    Load(response) {//Загрузка данных с API во VUE
      if (response.data.status) {
        // this.msg = response.data.msg;
        this.response = response.data.data;
      } else {
        this.msg = response.data.msg;
        this.errors = response.data.errors;
        this.snackbar = true;
        setTimeout(this.OutputClear, 5000);
      }
    },

    GetTagList() {//ЗАпрос на список тэгов с сервера
      // this.loadingTag = true;
      this.axios({
        method: "get",
        url: "https://cloud-vault.ru/api/tags",
        responseType: "json",
      })
        .catch((error) => {
          this.msg(error);
          this.snackbar = true;
        })
        .then((response) => {
          this.tags = response.data.data;
          // this.loadingTag=false;
        });
      // .finally(() => (this.loading = false));
    },
    OutputClear() {//Очистка вывода ошибок с сервера
      this.errors = null;
    },
    LoadPage(path) {//Загрузка постранички цитат
      this.loading = true;
      this.axios({
        method: "get",
        url: path,
        responseType: "json",
      })
        .then((response) => this.Load(response))
        .catch((error) => {
          this.msg(error);
          this.snackbar = true;
        })
        .finally(() => (this.loading = false));
    },
    PostQuote() {//Добавление цитаты в БД, через запрос
      if (this.postData.text && this.postData.author && this.postData.tags) {
        this.loading = true;
        this.axios
          .post("https://cloud-vault.ru/api/post", this.postData)
          .then((response) => this.Load(response))
          .catch((error) => {
            this.msg(error);
            this.snackbar = true;
          })
          .finally(() => (this.loading = false));
        this.dialog = false;
      } else {
        this.msg = "Заполните все поля!";
        this.snackbar = true;
      }
    },
  },
  mounted() {//Первичная загрузка данных
    this.loading = true;
    this.axios({
      method: "get",
      url: "https://cloud-vault.ru/api/list",
      responseType: "json",
    })
      .then((response) => this.Load(response))
      .catch((error) => {
        alert("Ошибка сервера!"+error);
      })
      .finally(() => (this.loading = false));
    this.GetTagList();
  },
  data: function () {
    return {
      response: null,//Данные от сервера
      valid: false,//Проверка валидации
      msg: null,//Текс сообщения от сервера
      errors: null,//Текст ошибок от сервера
      snackbar: false,//Отображение снэкбара
      timeout: 2000,//Задержка перед закрытием снэкбара
      drawer: false,
      loading: false,//Загрузка
      loadingTag: true,//Загрузка тэгов
      dialog: false,//Отображение модального окна
      RequiredRules: [(v) => !!v || "Обязательное поле"],//Required валидация
      MaxItemRules: [(v) => v.length <= 3 || "Не менее одного и не более трех"], //Проверка кол-ва элементов
      tags: [],//Список доступных тэгов
      postData: {//Объект для отправки на сервер
        text: null,//Текс цитаты
        author: null,//Автор
        tags: [],//Список тэгов
      },
    };
  },
  watch: {
    group() {
      this.drawer = false;
    },
  },
};
</script>
<style scoped>
</style>