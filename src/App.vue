<template>
  <v-app>
    <v-navigation-drawer
      absolute
      temporary
      v-model="drawer"
      class="hidden-md-and-up"
    >
      <v-list>
        <v-item-group>
          <v-item>
            <v-btn
              @click="dialog = !dialog"
              v-for="(item, i) in menuItems"
              text
              dense
              :key="`menuitem${i}`"
            >
              <v-icon left v-html="item.icon"></v-icon>
              {{ item.title }}</v-btn
            >
          </v-item>
        </v-item-group>
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
    <!-- <app-header></app-header> -->
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
              md="4"
            >
              <v-item>
                <v-card elevation="2">
                  <v-card-title>Автор: {{ item.author }}</v-card-title>
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

                  <v-card-text
                    ><strong>Текст:</strong> {{ item.text }}</v-card-text
                  >
                </v-card>
              </v-item>
            </v-col>
          </v-row>
          <v-row>
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
                  <!-- <validation-provider name="Name" rules="required|max:10"> -->
                  <v-textarea
                    v-model="postData.text"
                    label="Текст"
                    :rules="RequiredRules"
                    required
                    hint="Напишите текст вашей цитату сюда."
                  ></v-textarea>
                  <!-- </validation-provider> -->
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
    // TagName(){

    // },
  },

  methods: {
    BeautyDate(date) {
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
    Load(response) {
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

    GetTagList() {
      this.axios({
        method: "get",
        url: "http://127.0.0.1:8000/api/tags",
        responseType: "json",
      }).then((response) => (this.tags = response.data.data));
      // .finally(() => (this.loading = false));
    },
    OutputClear() {
      this.errors = null;
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
      if (this.postData.text && this.postData.author && this.postData.tags) {
        this.loading = true;
        this.axios
          .post("http://127.0.0.1:8000/api/post", this.postData)

          // ({
          //   method: "post",
          //   url: "http://127.0.0.1:8000/api/post",
          //   responseType: "json",
          // })
          .then((response) => this.Load(response))
          .finally(() => (this.loading = false));
        this.dialog = false;
      } else {
        this.msg = "Заполните все поля!";
        this.snackbar = true;
      }
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
      valid: false,
      msg: null,
      errors: null,
      snackbar: false,
      timeout: 2000,
      drawer: false,
      loading: false,
      dialog: false,
      RequiredRules: [
        (v) => !!v || "Обязательное поле",
      ],
      MaxItemRules:[
        v =>(v.length <= 3) || "Не менее одного и не более трех"

      ],
      tags: [],
      postData: {
        text: null,
        author: null,
        tags: [],
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