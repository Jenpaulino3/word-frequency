<template>
  <div class="quiz-questions">
    <p># of Questions Students Did Well On: {{this.questionsStudentsDidWellOn.length}}</p>
    <p># of Words Students Did Well With: {{this.wordsStudentsDidWellWith.length}} words</p>
    <p>{{this.sortedInstancesOfWordsStudentsDidWellWith}}</p>
    <hr />
    <p># of Questions Students Did Poorly On: {{this.questionsStudentsDidPoorlyOn.length}}</p>
    <p># of Words Students Did Poorly With: {{this.wordsStudentsDidPoorlyWith.length}} words</p>
    <p>{{this.sortedInstancesOfWordsStudentsDidPoorWith}}</p>
  </div>
</template>

<script>
import json from "../helpers/quizQuestions.json";
export default {
  name: "QuizQuestions",
  data() {
    return {
      quizQuestions: json.slice(0, 50),
      questionsStudentsDidPoorlyOn: [],
      questionsStudentsDidWellOn: [],
      wordsStudentsDidWellWith: [],
      wordsStudentsDidPoorlyWith: [],
      instancesOfWordsStudentsDidWellWith: [],
      instancesOfWordsStudentsDidPoorlyWith: [],
      sortedInstancesOfWordsStudentsDidWellWith: [],
      sortedInstancesOfWordsStudentsDidPoorWith: [],
      mostFrequentWellWords: null,
      frequentWellWords: null,
      mostFrequentPoorWords: null,
      frequentPoorWords: null
    };
  },
  mounted: function() {
    this.getQuestionsPerCondition("lessThanFifty", "greaterThanFifty");
    this.getWordsPerCondition(
      this.questionsStudentsDidWellOn,
      this.questionsStudentsDidPoorlyOn
    );
    this.getInstancesOfWordsPerCondition(
      this.wordsStudentsDidWellWith,
      this.wordsStudentsDidPoorlyWith
    );
    this.sortInstancesByDescendingOrder(
      this.instancesOfWordsStudentsDidWellWith,
      this.instancesOfWordsStudentsDidPoorlyWith
    );
    this.getSortedWellWordsAndInstances(
      this.instancesOfWordsStudentsDidWellWith,
      this.sortedInstancesOfWordsStudentsDidWellWith
    );
    this.getSortedPoorWordsAndInstances(
      this.instancesOfWordsStudentsDidPoorlyWith,
      this.sortedInstancesOfWordsStudentsDidPoorlyWith
    );
    this.getFrequencyOfWellWords(this.instancesOfWordsStudentsDidWellWith);
    this.getFrequencyOfPoorWords(this.instancesOfWordsStudentsDidPoorlyWith);
  },
  methods: {
    getQuestionsPerCondition(lessThanFifty, greaterThanFifty) {
      if (lessThanFifty) {
        this.questionsStudentsDidPoorlyOn = this.filterQuestionsByCondition(
          this.quizQuestions,
          lessThanFifty
        );
      }
      if (greaterThanFifty) {
        this.questionsStudentsDidWellOn = this.filterQuestionsByCondition(
          this.quizQuestions,
          greaterThanFifty
        );
      }
    },
    filterQuestionsByCondition(questions, condition) {
      return questions.filter(question => {
        if (condition === "lessThanFifty") {
          return question.percent_correct < 0.5;
        }

        if (condition === "greaterThanFifty") {
          return question.percent_correct > 0.5;
        }
      });
    },
    getWordsPerCondition(
      questionsStudentsDidWellOn,
      questionsStudentsDidPoorlyOn
    ) {
      let arr = [];
      if (questionsStudentsDidWellOn) {
        this.wordsStudentsDidWellWith = this.flattenAndFilterArrayOfWords(
          this.splitWordsAndPushToArray(questionsStudentsDidWellOn)
        );
      }

      if (questionsStudentsDidPoorlyOn) {
        this.wordsStudentsDidPoorlyWith = this.flattenAndFilterArrayOfWords(
          this.splitWordsAndPushToArray(questionsStudentsDidPoorlyOn)
        );
      }
    },
    splitWordsAndPushToArray(questions) {
      let arrayOfWords = [];
      questions.forEach(question => {
        arrayOfWords.push(question.text.split(" "));
      });
      return arrayOfWords;
    },
    flattenAndFilterArrayOfWords(arrayOfWords) {
      return arrayOfWords.flat().filter(e => e !== "");
    },
    getInstancesOfWordsPerCondition(
      wordsStudentsDidWellWith,
      wordsStudentsDidPoorlyWith
    ) {
      if (wordsStudentsDidWellWith) {
        this.instancesOfWordsStudentsDidWellWith = this.reduceArrayOfWords(
          wordsStudentsDidWellWith
        );
      }
      if (wordsStudentsDidPoorlyWith) {
        this.instancesOfWordsStudentsDidPoorlyWith = this.reduceArrayOfWords(
          wordsStudentsDidPoorlyWith
        );
      }
    },
    reduceArrayOfWords(words) {
      let instancesOfWords = words.reduce((accumulator, currentValue) => {
        accumulator[currentValue] = (accumulator[currentValue] || 0) + 1;
        return accumulator;
      }, {});
      return instancesOfWords;
    },
    sortInstancesByDescendingOrder(wellInstances, poorInstances) {
      if (wellInstances) {
        let sortedByDescending = Object.keys(wellInstances).sort((a, b) => {
          return wellInstances[b] - wellInstances[a];
        });
        this.sortedInstancesOfWordsStudentsDidWellWith = sortedByDescending;
      }
      if (poorInstances) {
        let sortedByDescending = Object.keys(poorInstances).sort((a, b) => {
          return poorInstances[b] - poorInstances[a];
        });
        this.sortedInstancesOfWordsStudentsDidPoorlyWith = sortedByDescending;
      }
    },
    getSortedWellWordsAndInstances(instances, sortedInstances) {
      let arr = [];
      sortedInstances.forEach(word => {
        for (let prop in instances) {
          return arr.push(`${word}: ${instances[word]}`);
        }
      });
      this.sortedInstancesOfWordsStudentsDidWellWith = arr;
    },
    getSortedPoorWordsAndInstances(instances, sortedInstances) {
      let arr = [];
      sortedInstances.forEach(word => {
        for (let prop in instances) {
          return arr.push(`${word}: ${instances[word]}`);
        }
      });
      this.sortedInstancesOfWordsStudentsDidPoorWith = arr;
    },
    getFrequencyOfWellWords(instances) {
      let mostFrequent = [];
      let frequent = [];
      for (let prop in instances) {
        if (instances[prop] >= 0.1 * this.wordsStudentsDidWellWith.length) {
          mostFrequent.push(instances[prop]);
        }
        if (
          instances[prop] < 0.1 * this.wordsStudentsDidWellWith.length &&
          instances[prop] > 0.02 * this.wordsStudentsDidWellWith.length
        ) {
          frequent.push(instances[prop]);
        }
      }
      this.mostFrequentWellWords = mostFrequent;
      this.frequentWellWords = frequent;
    },
    getFrequencyOfPoorWords(instances) {
      let mostFrequent = [];
      let frequent = [];
      for (let prop in instances) {
        if (instances[prop] >= 0.1 * this.wordsStudentsDidPoorlyWith.length) {
          mostFrequent.push(instances[prop]);
        }
        if (
          instances[prop] < 0.1 * this.wordsStudentsDidPoorlyWith.length &&
          instances[prop] > 0.02 * this.wordsStudentsDidPoorlyWith.length
        ) {
          frequent.push(instances[prop]);
        }
      }
      this.mostFrequentPoorWords = mostFrequent;
      this.frequentPoorWords = frequent;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>
