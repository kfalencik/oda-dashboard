<template>
  <section class="container">
    <h1>Statistics</h1>
    <div class="statistics" v-if="history.length">
      <div class="statistics__diagrams">
        <Diagram v-if="history.length" :title="'Queries made'" :total="history.length" :values="[customQueries(), history.length - customQueries()]" :labels="['Total queries made', 'Custom defined words', 'Randomly generated words']" />
        <Diagram v-if="totalWords" :title="'Queried words'" :total="totalWords" :values="[wordsWithResults(), totalWords - wordsWithResults()]" :labels="['Total words queried', 'Words with results', 'Words with no results']" />
      </div>

      <Statistic label="Total time spent querying" :value="`${totalTimeQuerying()}s`" />
      <Statistic label="Shortest query" :value="`${shortestQuery()}s`" />
      <Statistic label="Longest query" :value="`${longestQuery()}s`" />
      <Statistic label="Total results found" :value="`${totalResults}`" />
      <Statistic label="Most results per word" :value="`${mostResults()}`" />
      <Statistic label="Most searched word" :value="`${mostSearchedWord()}`" />
    </div>    

    <div v-else>
      <p>There are no statistics available at the moment. Please make some queries first and come back later.</p>
      <router-link class="button" to="/queries">Make a query</router-link>
    </div>
  </section>
</template>

<script>
import Diagram from '@/components/Diagram';
import Statistic from '@/components/Statistic';

export default {
  name: 'Statistics',
  components: {
    Diagram,
    Statistic
  },
  computed: {
    totalQueries() {
      return this.$store.state.statTotalQueries;
    },
    totalWords() {
      return this.$store.state.statTotalWords;
    },
    totalResults() {
      return this.$store.state.statTotalResults;
    },
    history() {
      return this.$store.state.history;
    }
  },
  methods: {
    wordsWithResults() {
      let words = 0;

      // Get all the words that have at least one result
      this.history.forEach(query => {
        query.words.forEach(word => {
          if (word.result.definitions.length) {
            words++;
          }
        })
      });
    
      return words;
    },
    customQueries() {
      let queries = 0;

      // Get queries that used user defined words
      this.history.forEach(query => {
        if (query.type === 'custom') {
          queries++;
        }
      });
    
      return queries;
    },
    totalTimeQuerying() {
      // Sum all queries time elapsed
      return [...this.history].reduce((accumulator, currentValue) => accumulator + currentValue.time, 0).toFixed(2);
    },
    shortestQuery() {
      // Find the shortest query
      return Math.min(...this.history.map(item => item.time)).toFixed(2);
    },
    longestQuery() {
      // Find the longest query
      return Math.max(...this.history.map(item => item.time)).toFixed(2);
    },
    mostResults() {
      // Find a word with most results
      return Math.max(...this.history.map(query => Math.max(...query.words.map(word => word.result.definitions.length))));
    },
    mostSearchedWord() {
      // Aggregate all searched words into one array
      const aggregatedResults = [];

      this.history.forEach(query => {
        aggregatedResults.push(...query.words.map(item => item.word));
      });

      // Find most occuring word in the aggregated results array
      return aggregatedResults.sort((a,b) => aggregatedResults.filter(v => v===a).length - aggregatedResults.filter(v => v===b).length).pop();
    }
  }
}
</script>

<style lang="scss" scoped>
  .statistics {
    &__diagrams {
      display: flex;
      flex-direction: column;

      @media (min-width: $screen-medium) {
        flex-direction: row;
      }
    }
  }
</style>