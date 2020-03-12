<template>
  <div id="app">
    <h4>{{ title }}</h4>
    <div class="form-group">
      <div class="container">
        <div class="row">
          <div class="col-2">
            <input
              type="button"
              @click="startNewGame"
              class="btn btn-success"
              v-if="inProgress==false"
              value="New"
            />
            <input
              type="button"
              @click="startNewGame"
              class="btn btn-success"
              v-else
              value="Restart"
            />
          </div>
          <div class="col-2">
            <input type="button" @click="shufflePack" class="btn btn-primary" value="Shuffle" />
          </div>
          <div class="col-3"></div>
        </div>
        <div class="row">
          <div class="col-12">
            <div class="player card bg-dark text-white" v-for="player in hands" :key="player">
              <div class="card-body">
                <h4 class="card-title">{{player.name}}</h4>
                <div class="form-group">
                  <div class="container">
                    <div
                      class="row"
                      v-bind:class="{ 'bust': player.isOver21, 'exactly21': player.is21 , 'stick': player.isSticking}"
                    >
                      <div class="col-12">
                        <div
                          class="playingCardContainer"
                          v-for="(item, index) in player.cards"
                          :key="index"
                        >
                          <img class="playingCard" :src="require(`@/assets/cards/${item}.png`)" />
                        </div>
                      </div>
                    </div>
                    <div class="row">
                      <div>Total: {{player.total}}</div>
                    </div>
                    <div class="row" v-if="!player.automated">
                      <div class="col-3">
                        <input
                          type="button"
                          :disabled="isLocked(player)"
                          @click="stick(player)"
                          class="btn btn-danger"
                          value="Stick"
                        />
                      </div>
                      <div class="col-3">
                        <input
                          type="button"
                          :disabled="isLocked(player)"
                          @click="twist(player)"
                          class="btn btn-success"
                          value="Twist"
                        />
                      </div>
                    </div>
                    <div class="row" v-else>
                      <div class="col-3">
                        <input
                          type="button"
                          :disabled="isLocked(player)"
                          @click="autoPlay(player)"
                          class="btn btn-success"
                          value="Play"
                        />
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "BlackJack",
  data() {
    return {
      cards: {
        numbers: [
          "2",
          "3",
          "4",
          "5",
          "6",
          "7",
          "8",
          "9",
          "10",
          "J",
          "Q",
          "K",
          "A"
        ],
        suits: ["H", "D", "S", "C"]
      },
      pack: [],
      title: "Black Jack",
      hands: {
        player1: {
          name: "Me",
          index: 0,
          cards: [],
          total: 0,
          is21: false,
          isOver21: false,
          isSticking: false,
          automated: false
        },
        player2: {
          name: "CPU",
          index: 1,
          cards: [],
          total: 0,
          is21: false,
          isOver21: false,
          isSticking: false,
          automated: true
        }
      },
      inProgress: false,
      turn: 0,
      playerCount: 2
    };
  },
  methods: {
    setupPack: function() {
      this.pack = [];
      var count = 0;
      for (var suitIndex in this.cards.suits) {
        for (var numberIndex in this.cards.numbers) {
          this.pack[count] =
            this.cards.suits[suitIndex] + this.cards.numbers[numberIndex];
          count++;
        }
      }
      this.shufflePack();
    },
    resetHands: function() {
      for (var playerIndex in this.hands) {
        var player = this.hands[playerIndex];
        player.cards = [];
        player.total = 0;
        player.isOver21 = false;
        player.is21 = false;
        player.isSticking = false;
      }
    },
    shufflePack: function() {
      var shuffledPack = [];
      var packSize = this.pack.length;
      for (var i = 0; i < packSize; i++) {
        var randomCardIndex = Math.floor(Math.random() * this.pack.length);
        shuffledPack[i] = this.pack[randomCardIndex];
        this.pack.splice(randomCardIndex, 1);
      }
      this.pack = shuffledPack;
    },
    startNewGame: function() {
      this.resetHands();
      this.setupPack();
      this.shufflePack();
      this.getHandTotals();
      this.inProgress = true;
      this.turn = 0;
    },
    dealCards: function(player, count = 1) {
      for (var i = 0; i < count; i++) {
        player.cards.push(this.pack[i]);
      }
      this.pack.splice(0, count);
      player.total = this.getHandTotal(player);
      player.isOver21 = player.total > 21;
      player.is21 = player.total == 21;
    },
    twist: function(player) {
      /*player.cards.push(this.pack[0]);
            this.pack.splice(0, 1);
            player.total = this.getHandTotal(player)
            player.isOver21 = (player.total > 21);
            player.is21 = (player.total == 21);*/
      this.dealCards(player);

      if (player.isOver21) this.endTurn();
    },
    stick: function(player) {
      player.isSticking = true;
      this.endTurn();
    },
    getHandTotals() {
      for (var player in this.hands) {
        this.dealCards(this.hands[player], 2);
        this.hands[player].total = this.getHandTotal(this.hands[player]);
      }
    },
    getHandTotal(player, lowAceCount = 0) {
      var total = 0;
      var aceCount = 0;
      for (var cardIndex in player.cards) {
        var cardNumber = player.cards[cardIndex].slice(1);
        cardNumber = cardNumber
          .replace("J", "10")
          .replace("Q", "10")
          .replace("K", "10");
        if (cardNumber == "A") {
          aceCount++;
        }
        if (aceCount > lowAceCount) {
          cardNumber = cardNumber.replace("A", "11");
        } else {
          cardNumber = cardNumber.replace("A", "1");
        }

        total += parseInt(cardNumber);
      }
      if (total > 21 && aceCount > lowAceCount) {
        return this.getHandTotal(player, lowAceCount + 1);
      }

      return total;
    },
    autoPlay: function(player) {
      if (player.total < 16) {
        this.twist(player);
        setTimeout(() => this.autoPlay(player), 1000);
      } else {
        this.stick(player);
      }
    },
    isLocked(player) {
      return (
        player.isOver21 ||
        player.isSticking ||
        !this.inProgress ||
        this.turn != player.index
      );
    },
    showResult() {
      var highScore = 0;
      var winningPlayerNames='';
      var draw = false;
      for (var playerIndex in this.hands) {
        var player = this.hands[playerIndex];
        if (!player.isOver21) {
          if (player.total > highScore) {
            highScore = player.total;
            winningPlayerNames = player.name;
            draw = false;
          } else if (player.total == highScore) {
            winningPlayerNames += ", " + player.name;
            draw = true;
          }
        }
      }

      var msg;
      if (draw) {
        msg = winningPlayerNames + " win";
      } else {
        msg = winningPlayerNames + " wins";
      }
      setTimeout(() => alert(msg), 1000);
    },
    endTurn() {
      if (this.turn == Object.keys(this.hands).length - 1) {
        this.showResult();
      } else {
        this.turn++;
      }
    }
  },
  mounted: function() {
    this.setupPack();
    this.resetHands();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#app {
  padding: 50px;
}

#regForm {
  background-color: #ffffff;
  margin: 100px auto;
  padding: 40px;
  width: 70%;
  min-width: 300px;
}

input {
  padding: 10px;
  width: 100%;
  font-size: 17px;
  font-family: arial;
  border: 1px solid #aaaaaa;
}

input.invalid {
  background-color: #ffdddd;
}

.form-group .container .row {
  padding: 5px;
}

label {
  display: block;
}

#qrPreview {
  height: 200px;
  width: 200px;
}

.urlInput.valid {
  background-color: rgb(228, 255, 234);
}

.urlInput.invalid {
  background-color: pink;
}

.bust {
  background-color: red;
}

.exactly21 {
  background-color: green;
}

.player {
  margin: 15px;
}

.stick {
  background-color: grey;
}

.playingCard {
  background-color: white;
  width: 80px;
}

.playingCardContainer {
  float: left;
  padding-right: 5px;
}
</style>
