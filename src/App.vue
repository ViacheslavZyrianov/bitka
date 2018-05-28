<template>
  <div class="game">
    <template v-if="state === 'start'">
      <h1>BITKA</h1>
      <h2>Hello, {{heroName || 'summoner'}}!</h2>
      <h3>Enter your name:</h3>
      <input type="text" placeholder="Your name..." maxlength="64" v-model="heroName">
      <h2>Press start to begin!</h2>
      <button
        :disabled="isBtnStartDisabled"
        class="btn btn-start"
        @click="startGame"
        >Start
      </button>
    </template>
    <template v-if="state === 'game' || state === 'finish'">
      <div class="players">
        <div class="player hero">
          <div class="player-name">{{ heroName }}</div>
          <div class="player-pic" :style="{'background-image': 'url('+heroPic+')'}"></div>
          <div class="hp-bar">
            <div class="hp" :style="{'transform': 'scaleX('+heroHP/100+')'}"></div>
            <div class="hp-num">{{heroHP}}</div>
          </div>
        </div>
        <div class="player enemy">
          <div class="player-name">Enemy</div>
          <div class="player-pic" :style="{'background-image': 'url('+enemyPic+')'}"></div>
          <div class="hp-bar">
            <div class="hp" :style="{'transform': 'scaleX('+enemyHP/100+')'}"></div>
            <div class="hp-num">{{enemyHP}}</div>
          </div>
        </div>
      </div>
      <div class="buttons">
        <template v-if="state === 'finish'">
          <button class="btn btn-restart" @click="restart">Restart</button>
          <button class="btn btn-exit" @click="exit">Exit</button>
        </template>
        <template v-else>
          <button class="btn btn-attack" @click="attack" :disabled="!isAttackEnabled">Attack</button>
          <!-- <button class="btn btn-special-attack">Special attack</button> -->
          <button class="btn btn-heal" @click="heal" :disabled="isHealDisabled">Heal</button>
          <button class="btn btn-exit" @click="exit">Exit</button>
        </template>
      </div>
      <ul class="battle-log">
        <li
            class="log"
            :class="`log-${battleLogItem.type}`"
            :key="i"
            v-for="(battleLogItem, i) in battleLog">
            {{battleLogOutput(battleLogItem)}}
        </li>
      </ul>
    </template>
  </div>
</template>

<script>
import random from './utils/random';

export default {
  name: 'app',
  data() {
    return {
      state: 'start',
      isAttackEnabled: true,
      isHealDisabled: true,
      heroName: '',
      heroPic: null,
      enemyPic: null,
      heroHP: 0,
      enemyHP: 0,
      heroAttackedOn: 0,
      enemyAttackedOn: 0,
      currentHeroAction: '',
      battleLog: []
    };
  },
  methods: {
    startGame () {
      this.clearData();
      this.state = 'game';
      this.heroPic = `src/img/players/${random(0, 95)}.svg`;
      this.enemyPic = `src/img/players/${random(0, 95)}.svg`;
      this.heroHP = 100;
      this.enemyHP = 100;
    },
    restart () {
      this.clearData();
      this.startGame();
    },
    exit () {
      this.state = 'start';
    },
    clearData () {
      this.battleLog = [];
      this.heroPic = this.enemyPic = null;
      this.heroHP = this.enemyHP = 0;
    },
    attackCount () {
      this.heroAttackedOn = random(0, 20);
      this.enemyAttackedOn = random(0, 20);
    },
    attack () {
      this.currentHeroAction = 'attack';
      this.attackCount();
      this.heroHP -= this.enemyAttackedOn;
      this.battleLogUpdate('heroAttack', this.heroAttackedOn);
      this.enemyHP -= this.heroAttackedOn;
      this.battleLogUpdate('enemyAttack', this.enemyAttackedOn);
    },
    heal () {
      const heroHealHP = this.heroHealedOn;
      this.currentHeroAction = 'heal';
      this.heroHP += heroHealHP;
      this.battleLogUpdate('heal', heroHealHP);
    },
    finish (winner) {
      this.state = 'finish';
      this.battleLogUpdate('finish', winner);
    },
    battleLogUpdate (type, param) {
      this.battleLog.unshift({
        type: type,
        param: param
      });
    },
    battleLogOutput(battleLogItem) {
      switch (battleLogItem.type) {
        case 'draw': return `It's draw!`;
        case 'heroAttack': return `${this.heroName} attacked on ${battleLogItem.param} HP!`;
        case 'enemyAttack': return `Enemy attacked on ${battleLogItem.param} HP!`;
        case 'heal': return `${this.heroName} healed on ${battleLogItem.param} HP!`;
        case 'finish': return `${battleLogItem.param} wins!`;
      }
    }
  },
  computed: {
    heroHealedOn () {
      const healOn = random(1,10);
      return healOn >= 100 - this.heroHP ? 100 - this.heroHP : healOn;
    },
    isBtnStartDisabled () {
      return this.heroName.length < 2
    }
  },
  watch: {
    heroHP () {
      this.heroHP <= 0 ? this.finish('Enemy') : this.isHealDisabled = this.heroHP >= 100
    },
    enemyHP () {
      if (this.enemyHP <= 0) this.finish(this.heroName);
    }
  }
};
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  padding: 0;
}

ul {
  padding: 0;
  margin: 0;
}

.game {
  display: block;
  margin: auto;
  font-family: "Helvetica", sans-serif;
  max-width: 1200px;
  text-align: center;
}

.players {
  width: 100%;
  display: flex;

  .player {
    width: 50%;

    .player-name {
      margin: 16px 0;
    }

    .player-pic {
      position: relative;
      margin: 0 auto 16px;
      width: 300px;
      height: 300px;
      background-position: center center;
      background-size: contain;
      background-repeat: no-repeat;
    }

    .hp-bar {
      position: relative;
      margin: 0 auto 16px;
      width: 80%;
      height: 32px;
      border: 2px solid rgb(94, 94, 94);
      background-color: rgb(192, 192, 192);
      overflow: hidden;

      .hp {
        position: relative;
        width: 100%;
        height: 100%;
        background-color: green;
        transition: transform 0.3s;
        transform: scaleX(1);
        transform-origin: left center;
      }

      .hp-num {
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        margin: auto;
        width: 50px;
        height: 18px;
        color: #fff;
      }
    }
  }
}

.battle-log {
  position: relative;
  width: 90%;
  margin: auto;
  height: 400px;
  overflow-y: auto;

  .log {
    width: 100%;
    margin-bottom: 8px;
    font-weight: bold;
    color: #fff;
    padding: 12px 0;

    &.log-heroAttack {
      background-color: green;
    }

    &.log-enemyAttack {
      background-color: red;
    }

    &.log-heal {
      background-color: hotpink;
    }

    &.log-finish {
      background-color: orange;
    }
  }
}

.buttons {
  position: relative;
  margin-bottom: 16px;
}

.btn {
  padding: 8px 16px;
  border: 0;
  border-radius: 0;
  color: #fff;
  user-select: none;
  cursor: pointer;

  &.btn-start {
    background-color: green;
  }
  &.btn-attack {
    background-color: red;
  }
  &.btn-special-attack {
    background-color: orange;
  }
  &.btn-heal {
    background-color: green;
  }
  &.btn-restart {
    background: blue;
  }
  &.btn-exit {
    background-color: cornflowerblue;
  }
  &:focus {
    outline: none;
  }
  &[disabled] {
    background-color: gray;
    color: red;
    cursor: not-allowed;
  }
}
</style>
