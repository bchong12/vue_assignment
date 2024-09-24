<template>
  <div>
    <p>{{ loading_status }}</p>
    <h3>Jeopardy Game</h3>
    <h1>{{winningMessage}}</h1>
    <div id="questionsDiv">
      <!-- Category 1 -->
      <div class="questions">
        <h4 class = "cat_header">{{ cat_headers[0] }}</h4>
        <p v-for="(question, index) in questions[0]" :key="index">
          <!-- Conditionally show dollar value or question -->
          <span v-if="!revealedQuestions[0][index]" @click="revealQuestion(0, index)">
            ${{ getDollarValue(index) }}
          </span>
          <span v-else :class="questionStates[0][index]">
            {{ question.question }}
          </span>
        </p>
      </div>
      
      <!-- Category 2 -->
      <div class="questions">
        <h4 class = "cat_header">{{ cat_headers[1] }}</h4>
        <p v-for="(question, index) in questions[1]" :key="index">
          <span v-if="!revealedQuestions[1][index]" @click="revealQuestion(1, index)">
            ${{ getDollarValue(index) }}
          </span>
          <span v-else :class="questionStates[1][index]">
            {{ question.question }}
          </span>
        </p>
      </div>

      <!-- Category 3 -->
      <div class="questions">
        <h4 class = "cat_header">{{ cat_headers[2] }}</h4>
        <p v-for="(question, index) in questions[2]" :key="index">
          <span v-if="!revealedQuestions[2][index]" @click="revealQuestion(2, index)">
            ${{ getDollarValue(index) }}
          </span>
          <span v-else :class="questionStates[2][index]">
            {{ question.question }}
          </span>
        </p>
      </div>

      <!-- Category 4 -->
      <div class="questions">
        <h4 class = "cat_header">{{ cat_headers[3] }}</h4>
        <p v-for="(question, index) in questions[3]" :key="index">
          <span v-if="!revealedQuestions[3][index]" @click="revealQuestion(3, index)">
            ${{ getDollarValue(index) }}
          </span>
          <span v-else :class="questionStates[4][index]">
            {{ question.question }}
          </span>
        </p>
      </div>
    </div>

    <div id="userDiv">
      <h4>Current Question for Player {{current_player + 1}}</h4>
      <p>{{currentQuestion}}</p>
      <div id="userButtons">
        <button id="userButtonTrue" @click="adjustPlayer('True')">True</button>
        <button @click="adjustPlayer('False')">False</button>
      </div>
    </div>

    <div id="playerAmountDiv">
    <h4>Players</h4>
      <p v-for="(amount, index) in player_amount" :key = "index">Player {{index + 1}}: ${{amount}}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Jeopardy_game',
  data() {
    return {
      categories: [],
      cat_headers: [],
      questions: [[], [], [], []],
      questionStates:[[null, null, null, null, null],[null, null, null, null, null],[null, null, null, null, null],[null, null, null, null, null]],
      currentQuestion: '',
      currCategoryIndex: 0,
      currentIndex: 0,
      currentAnswer: '',
      player_amount: [0, 0, 0],
      current_player: 0,
      revealQuestionStatus: true,
      winningMessage: '',
      category_choices: {
        9: true,
        11: true,
        12: true,
        14: true,
        15: true,
        16: true,
        17: true,
        19: true,
        20: true,
        22: true,
        23: true,
        24: true,
        25: true,
        28: true,
        31: true
      },
      loading_status: 'API Loading... (please wait till it loads [around 30 seconds..])',
      revealedQuestions: [
        [false, false, false, false, false], // Initial state for questions in category 1
        [false, false, false, false, false], // Initial state for questions in category 2
        [false, false, false, false, false], // Initial state for questions in category 3
        [false, false, false, false, false], // Initial state for questions in category 4
      ],
    }
  },
  mounted() {

  this.categories = this.getCategories().sort((a, b) => a - b);
  fetch(`https://opentdb.com/api_category.php`)
    .then(response => response.json())
    .then(data => {
      let res = data.trivia_categories;
      
      for (let i = 0; i < res.length; i++) {
        if(this.categories.includes(res[i].id)) {
          this.cat_headers.push(res[i].name);
        }
      }
    })
    .catch(error => {
      console.error('Error fetching data:', error);
    });

  const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

  const fetchWithDelay = async () => {

    for (let i = 0; i < this.categories.length; i++) {
      await delay(6000);
      fetch(`https://opentdb.com/api.php?amount=2&category=${this.categories[i]}&difficulty=easy&type=boolean`)
        .then(response => response.json())
        .then(data => {
          this.questions[i].push(data.results[0]);
          this.questions[i].push(data.results[1]);
        })
        .catch(error => {
          console.error('Error fetching data:', error);
        });
       
    }

    for (let i = 0; i < this.categories.length; i++) {
      await delay(6000);
      fetch(`https://opentdb.com/api.php?amount=2&category=${this.categories[i]}&difficulty=medium&type=boolean`)
        .then(response => response.json())
        .then(data => {
          this.questions[i].push(data.results[0]);
          this.questions[i].push(data.results[1]);
        })
        .catch(error => {
          console.error('Error fetching data:', error);
        });
       
    }

    for (let i = 0; i < this.categories.length; i++) {
      await delay(6000);
      fetch(`https://opentdb.com/api.php?amount=1&category=${this.categories[i]}&difficulty=hard&type=boolean`)
        .then(response => response.json())
        .then(data => {
          this.questions[i].push(data.results[0]);
        })
        .catch(error => {
          console.error('Error fetching data:', error);
        });
       
    }
    this.loading_status = '';
  };

  fetchWithDelay();
},
  methods: {
    getCategories() {
      let keys = Object.keys(this.category_choices);
      let shuffled = keys.sort(() => 0.5 - Math.random());
      shuffled = shuffled.slice(0, 4);
      return shuffled.map(Number);
    },
    getDollarValue(index) {
      const dollarValues = [100, 200, 300, 400, 500];
      return dollarValues[index];
    },
    revealQuestion(categoryIndex, questionIndex) {
      if(this.revealQuestionStatus) {
        this.revealedQuestions[categoryIndex][questionIndex] = true;
        this.currentQuestion = this.questions[categoryIndex][questionIndex].question;
        this.currentAnswer = this.questions[categoryIndex][questionIndex].correct_answer;
        this.currentIndex = questionIndex;
        this.currCategoryIndex = categoryIndex;
        this.revealQuestionStatus = false;
      }
    },
    adjustPlayer(answer) {
        if(this.revealQuestionStatus === false) {
          if(answer === this.currentAnswer) {
            this.player_amount[this.current_player] += parseInt(this.getDollarValue(this.currentIndex));
            this.questionStates[this.currCategoryIndex][this.currentIndex] = 'highlight-correct';
          } else {
            this.player_amount[this.current_player] -= parseInt(this.getDollarValue(this.currentIndex));
            this.questionStates[this.currCategoryIndex][this.currentIndex] = 'highlight-incorrect';
          }
          this.questions[this.currCategoryIndex][this.currentIndex].question = 'P' + parseInt(this.current_player + 1);
          this.current_player = (this.current_player + 1) % 3;
          this.revealQuestionStatus = true;

          if (this.areAllQuestionsRevealed()) {
            let max = Math.max(...this.player_amount);
            let index = this.player_amount.indexOf(max);
            this.winningMessage = `Player ${index + 1} wins with $${max}!`;
          }
        }
    },
    areAllQuestionsRevealed() {
        const allRevealed = this.revealedQuestions.every(category => 
    category.every(revealed => revealed === true)
      );
      
      return allRevealed;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.cat_header {
  margin: 20px 100px;
}

.questions {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  margin: 20px 100px;
}
#questionsDiv {
  display: flex;
  align-items: center;
  justify-content: center;
}
#playerAmountDiv {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  flex-direction: column;
  margin-left: 300px;
  border: 1px solid black;
  width: fit-content;
  padding: 0px 50px;
  margin-top: 100px;
}
#userDiv {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  flex-direction: column;
  margin-left: 300px;
  width: fit-content;
  padding: 0px 50px;
  margin-top: 300px;
 
}
#userButtonTrue {
  margin-right:10px;
}
.highlight-correct {
  color: green; /* Color for correct answers */
}
.highlight-incorrect {
  color: red; /* Color for incorrect answers */
}
</style>
