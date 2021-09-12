<template>
  <v-app>
    <v-navigation-drawer absolute temporary v-model="drawer">
      <v-list></v-list>
    </v-navigation-drawer>
    <v-app-bar app dark>
      <v-text-field v-text="'Quotes'"></v-text-field>
      <v-spacer></v-spacer>
      <v-item-group>
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
    <v-main>
      <v-item-group>
        <v-container>
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
              md="4"
            >
              <v-item>
                <v-card elevation="2">
                  <v-card-title>Автор: {{ item.author }}</v-card-title>
                  <v-card-subtitle>{{ item.created_at }}</v-card-subtitle>
                  <v-card-subtitle>{{ item.tags.name }}</v-card-subtitle>
                  <v-card-text>{{ item.text }}</v-card-text>
                </v-card>
              </v-item>
            </v-col>
          </v-row>
          <v-row>
            <v-toolbar dense flat>
              <v-toolbar-items>
                <v-btn
                  @click="LoadPage(response.prev_page_url)"
                  text
                  flat
                  v-show="response.prev_page_url"
                  >Предыдущая страница</v-btn
                >
                <v-spacer></v-spacer>
                <v-btn
                  @click="LoadPage(response.next_page_url)"
                  text
                  flat
                  left
                  v-show="response.next_page_url"
                  >Следующая страница</v-btn
                >
              </v-toolbar-items>
              <v-toolbar-title left
                >Страница: {{ response.current_page }}/{{
                  response.last_page
                }}</v-toolbar-title
              >
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
    <!-- MODAL INPUT -->
    <v-dialog v-model="dialog" width="500">
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
                  hint="Ваше имя, псевдоним или прозвище."
                ></v-text-field>
              </v-col>

              <v-col cols="12" sm="6">
                <v-autocomplete
                  v-model="postData.tags"
                  :items="[
                    'Skiing',
                    'Ice hockey',
                    'Soccer',
                    'Basketball',
                    'Hockey',
                    'Reading',
                    'Writing',
                    'Coding',
                    'Basejump',
                  ]"
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
          <v-btn color="blue darken-1" text @click="dialog = false">
            Добавить
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
// import func from "vue-editor-bridge";
export default {
  computed: {
    menuItems() {
      return [
        {
          icon: "mdi-pencil",
          title: "Добавить цитату",
        },
      ];
    },
  },

  methods: {
    Load(response) {
      if (response.data.status) {
        this.msg = response.data.msg;
        this.response = response.data.data;
        this.snackbar = true;
      }
    },
    LoadPage(path) {
      this.loading = true;
      this.axios({
        method: "get",
        url: path,
        responseType: "json",
      })
        .then((response) => this.Load(response))
        .finally(() => (this.loading = false));
    },
    PostQuote() {
      this.axios({
        method: "post",
        url: "http://127.0.0.1:8000/api/post",
        responseType: "json",
      })
        .then((response) => this.Load(response))
        .finally(() => (this.loading = false));
    },
  },
  mounted() {
    this.loading = true;
    this.axios({
      method: "get",
      url: "http://127.0.0.1:8000/api/list",
      responseType: "json",
    })
      .then((response) => this.Load(response))
      .finally(() => (this.loading = false));
  },
  data: function () {
    return {
      response: null,
      msg: null,
      snackbar: false,
      timeout: 2000,
      drawer: false,
      loading: false,
      dialog: false,
      postData: {
        text: null,
        author: null,
        tags: [],
      },
    };
  },
};
</script>
<style scoped>
</style>