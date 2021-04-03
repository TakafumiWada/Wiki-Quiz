<template>
  <div class="home">
    <button @click="clickStartButton">取得</button>
    <div></div>
    <div v-if="startQuiz">
      <div v-if="isLoading"><LoadingComponent /></div>
      <div v-else><QuizContent /></div>
    </div>
  </div>
</template>

<script>
import LoadingComponent from "@/components/Loading.vue";
import QuizContent from "@/components/QuizContent.vue";

export default {
  name: "Home",
  components: {
    LoadingComponent,
    QuizContent,
  },
  data() {
    return {
      quizData: {
        pageId: 0,
        title: "",
        text: "",
        keyWord: [],
        categories: [],
      },
      startQuiz: false,
      isLoading: true,
    };
  },
  methods: {
    async clickStartButton() {
      this.startQuiz = true;
      this.isLoading = true;
      while (this.validQuestion()) {
        await this.getQuestion();
      }
      this.isLoading = false;
      console.log(this.quizData.keyWord);
      console.log(this.quizData.categories);
    },
    getQuestion() {
      this.axios({
        url:
          "https://ja.wikipedia.org/w/api.php?format=json&action=query&generator=random&grnnamespace=0&prop=revisions&rvprop=content&indexpageids",
        method: "GET",
        withCredentials: true,
      }).then((res) => {
        this.initData();
        this.setQuizData(res.data);
        this.getKeyWord(this.quizData.text);
        console.log(this.quizData.keyWord);
        console.log(this.quizData.categories);
      });
    },
    initData() {
      this.quizData.pageId = 0;
      this.quizData.title = "";
      this.quizData.text = "";
      this.quizData.keyWord = [];
      this.quizData.categories = [];
    },
    setQuizData(data) {
      const id = data.query.pageids[0];
      this.quizData.pageId = id;
      this.quizData.title = data.query.pages[id].title;
      this.quizData.text = data.query.pages[id].revisions[0]["*"];
    },
    getKeyWord(text) {
      const array = text.split("");
      let isLink = false;
      let isDup = false;
      let word = "";
      this.quizData.keyWord = [];
      for (let i = 1; i < array.length - 1; i++) {
        if (array[i] == "]") {
          if (array[i + 1] == "]") {
            isLink = false;
            if (word.match(/Category:/)) {
              word = word.split("Category:").join("");
              for (let t = 0; t < this.quizData.categories; t++) {
                if (this.quizData.categories[t] === word) {
                  isDup = true;
                }
              }
              if (!isDup) {
                this.quizData.categories.push(word);
              }
            } else {
              for (let j = 0; j < this.quizData.keyWord; j++) {
                if (this.quizData.keyWord[j] === word) {
                  isDup = true;
                }
              }
              if (!isDup) {
                this.quizData.keyWord.push(word);
              }
              isDup = false;
            }
            word = "";
          }
        }
        if (isLink) {
          word = word.concat(array[i]);
        }
        if (array[i] == "[") {
          if (array[i - 1] == "[") {
            isLink = true;
          }
        }
      }
    },
    validQuestion() {
      return (
        this.quizData.keyWord.length > 10 && this.quizData.categories.length > 4
      );
    },
  },
};
</script>
