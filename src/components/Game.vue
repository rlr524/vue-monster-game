<template>
  <div id="app">
    <section class="row">
      <div class="small-6 columns">
        <h1 class="text-center">YOU</h1>
        <div class="healthbar">
          <div
            class="healthbar text-center"
            style="background-color: green; margin: 0; color: white; font-weight: bold"
            :style="{ width: playerHealth + '%' }"
          >
            {{ playerHealth }}
          </div>
        </div>
      </div>
      <div class="small-6 columns">
        <h1 class="text-center">MONSTER</h1>
        <div class="healthbar">
          <div
            class="healthbar text-center"
            style="background-color: green; margin: 0; color: white; font-weight: bold"
            :style="{ width: monsterHealth + '%' }"
          >
            {{ monsterHealth }}
          </div>
        </div>
      </div>
    </section>
    <!-- v-if and v-else need to follow each other and both need to be on the same element (e.g. both on a section as here) -->
    <section class="row controls" v-if="!gameIsRunning">
      <div class="small-12 columns">
        <button id="start-game" @click="startGame">START NEW GAME</button>
      </div>
    </section>
    <section class="row controls" v-else>
      <div class="small-12 columns">
        <button id="attack" @click="attack">ATTACK</button>
        <button id="special-attack" @click="specialAttack">
          SPECIAL ATTACK
          <span class="button-count" v-if="specialAttacks >= 0">
            {{ specialAttacks }}
          </span>
          <span class="button-count" v-else>0</span>
        </button>
        <button id="heal" @click="heal">
          HEAL
          <span class="button-count" v-if="heals >= 0">{{ heals }}</span>
          <span class="button-count" v-else>0</span>
        </button>
        <button id="give-up" @click="giveUp">GIVE UP</button>
      </div>
    </section>
    <section class="row log" v-if="turns.length != 0">
      <div class="small-12 columns">
        <ul>
          <li
            v-for="(turn, index) in turns"
            :key="index"
            :class="{
              'player-turn': turn.isPlayer,
              'monster-turn': !turn.isPlayer
            }"
          >
            {{ turn.text }}
          </li>
        </ul>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  name: "Game",
  data() {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      gameIsRunning: false,
      specialAttacks: 3,
      heals: 3,
      turns: []
    };
  },
  components: {},
  methods: {
    startGame: function() {
      this.turns = [];
      this.gameIsRunning = true;
      this.playerHealth = 100;
      this.monsterHealth = 100;
      this.specialAttacks = 3;
      this.heals = 3;
    },
    attack: function() {
      let damage = this.calculateDamage(3, 10);
      this.monsterHealth -= damage;
      this.turns.unshift({
        isPlayer: true,
        text: `Player hits monster for ${damage}`
      });
      if (this.checkWin()) {
        return;
      }
      this.monsterAttacks();
    },
    specialAttack: function() {
      this.specialAttacks--;
      let damage = this.calculateDamage(10, 20);
      if (this.specialAttacks >= 0) {
        this.monsterHealth -= damage;
        this.turns.unshift({
          isPlayer: true,
          text: `Player special attacks monster for ${damage}`
        });
        if (this.checkWin()) {
          return;
        }
        this.monsterAttacks();
      } else {
        document.getElementById("special-attack").style = "disabled";
      }
    },
    monsterAttacks: function() {
      let damage = this.calculateDamage(5, 12);
      this.playerHealth -= damage;
      this.checkWin();
      this.turns.unshift({
        isPlayer: false,
        text: `Monster hits player for ${damage}`
      });
    },
    heal: function() {
      if (this.heals >= 0 && this.playerHealth <= 90) {
        let damage = 10;
        this.heals--;
        this.playerHealth += damage;
        this.monsterAttacks();
        this.turns.unshift({
          isPlayer: true,
          text: `Player heals ${damage} of damage`
        });
      } else if (this.heals >= 0 && this.playerHealth >= 91) {
        alert("You cannot use healing if your health is over 90");
        document.getElementById("heal").style = "disabled";
      } else {
        document.getElementById("heal").style = "disabled";
      }
    },
    giveUp: function() {
      this.gameIsRunning = false;
      this.turns.unshift({
        isPlayer: true,
        text: `Player gives up`
      });
    },
    calculateDamage: function(minDamage, maxDamage) {
      return Math.max(Math.floor(Math.random() * maxDamage) + 1, minDamage);
    },
    checkWin: function() {
      if (this.monsterHealth <= 0) {
        if (confirm("You won! New game?")) {
          this.startGame();
        } else {
          this.gameIsRunning = false;
        }
        return true;
      } else if (this.playerHealth <= 0) {
        if (confirm("You're dead! New game?")) {
          this.startGame();
        } else {
          this.gameIsRunning = false;
        }
        return true;
      }
      return false;
    }
  },
  computed: [],
  props: {}
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.button-count {
  font-weight: 700;
  color: white;
  border: 1px;
  background-color: black;
  padding: 0 3px;
}
</style>
