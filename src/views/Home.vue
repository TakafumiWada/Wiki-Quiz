<template>
  <div class="home">
    <button @click="getQuestion">取得</button>
    <div>{{ quizData }}</div>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      quizData: {
        pageId: 0,
        title: "",
        text: "",
        keyWord: [],
        categories: [],
      },
    };
  },
  methods: {
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
        if (!this.validQuestion()) {
          this.getQuestion();
        }
        console.log(this.quizData);
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
