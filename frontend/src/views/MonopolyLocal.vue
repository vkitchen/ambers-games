<template>
  <div class="game-page">
    <h1>
      <img src="../assets/flower.png" style="height:30px;">
      <router-link :to="{name: 'Home'}" class="link__none-style">Monopoly</router-link>
    </h1>

    <div class="game-content competition">
      <div>
        <div class="game-round">Round {{round}}</div>
        <div v-if="winner != ''" class="game-congratulation">Congradulation on {{winner}}!</div>
        <table class="playerList">
          <tr>
            <th class="playList__playerNow"></th>
            <th>Player</th>
            <th>Coin</th>
            <th>Grass in hand</th>
            <th>Blocks</th>
          </tr>
          <tr v-for="(playerInfo, index) in playerList" :key="index">
            <td
              class="playList__playerNow"
              v-bind:class="{
              'player1-land ': index == 0,
              'player2-land_title': index == 1,
              'player3-land_title': index == 2,
              'player4-land_title': index == 3,
              }"
            >
              <div v-if="index == playerNowIndex">
                <img src="../assets/fingerPoint.png" alt="this" class="pixelart">
              </div>
            </td>
            <td>{{playerInfo.playerId}}</td>
            <td>${{playerInfo.coinNum}}</td>
            <td>{{playerInfo.grassNum}}</td>
            <td>{{playerInfo.blockNum}}</td>
          </tr>
        </table>
        <br>

        <div v-if="step == 1">
          <label>
            Please choose a direction:
            <select
              v-model="directionChoose"
              v-on:change="calcDirection()"
            >
              <option value="forward">forward</option>
              <option value="left">left</option>
              <option value="right">right</option>
            </select>
          </label>
          <button v-on:click="directionDecide()" class="button-inline">Decide</button>
        </div>

        <div v-if="step == 2" class="diceNshaker">
          <img
            src="../assets/diceShaker.png"
            alt="dice shaker"
            class="diceShaker pixelart"
            v-bind:class="{'shakerShake': shaking, 'shakerOpen': opening}"
          >
          <img v-if="diceShow == 1" src="../assets/dice1.png" alt="dice 1" class="dice pixelart">
          <img
            v-else-if="diceShow == 2"
            src="../assets/dice2.png"
            alt="dice 2"
            class="dice pixelart"
          >
          <img
            v-else-if="diceShow == 3"
            src="../assets/dice3.png"
            alt="dice 3"
            class="dice pixelart"
          >
          <img
            v-else-if="diceShow == 4"
            src="../assets/dice4.png"
            alt="dice 4"
            class="dice pixelart"
          >
          <img
            v-else-if="diceShow == 5"
            src="../assets/dice5.png"
            alt="dice 5"
            class="dice pixelart"
          >
          <img
            v-else-if="diceShow == 6"
            src="../assets/dice6.png"
            alt="dice 6"
            class="dice pixelart"
          >

          <button v-if="!shaking" v-on:click="shakeShaker()">Shake</button>
          <button v-else-if="!opening" v-on:click="openShaker()">Open</button>
        </div>

        <div v-if="step == 3 && payment != 0">
          You have to pay ${{payment}} to {{playerPayTo}}.
          <button v-on:click="pay()">Pay</button>
        </div>

        <div v-if="step == 4">
          Please choose two connective pieces to put the grass:
          <br>
          <button v-on:click="placeGrass()" class="button-inline">Decide</button>
          <button v-on:click="resetGrassChoose()" class="button-inline">Reset</button>
          <button v-on:click="skipGrassChoose()" class="button-inline">Skip</button>
        </div>
      </div>

      <div class="board-background-img">
        <div class="board">
          <table>
            <tr v-for="(line, indexLine) in board" :key="indexLine">
              <td
                v-for="(cell, indexCell) in line"
                :key="indexCell"
                class="game__cell"
                v-on:click="chooseCell(indexLine, indexCell)"
                v-bind:class="{
              'player1-land pixelart': board[indexLine][indexCell] == 'player1',
              'player2-land pixelart': board[indexLine][indexCell] == 'player2',
              'player3-land pixelart': board[indexLine][indexCell] == 'player3',
              'player4-land pixelart': board[indexLine][indexCell] == 'player4',
              }"
              >
                <div
                  v-if="(indexLine == grassPieceOnei && indexCell == grassPieceOnej) || (indexLine == grassPieceTwoi && indexCell == grassPieceTwoj)"
                  v-bind:class="{
                  'cell-chosen__player1': playerNow == 'player1',
                  'cell-chosen__player2': playerNow == 'player2',
                  'cell-chosen__player3': playerNow == 'player3',
                  'cell-chosen__player4': playerNow == 'player4'
              }"
                ></div>
              </td>
            </tr>
          </table>

          <img
            v-if=" textShow == true "
            src="../assets/wheretoputgrass.png"
            alt="where to put the grass?"
            class="pixelart text-wrapper"
            :style="characterPosition"
          >
          <img
            v-if=" errorMessage == 'Grasses are used up'"
            src="../assets/nograssleft.png"
            alt="You have used up all the grass, click skip to continue"
            class="pixelart text-two-line-wrapper"
            :style="characterPosition"
          >
          <img
            v-if=" errorMessage == 'You need to choose two pieces'"
            src="../assets/youneedtochoosetwopiece.png"
            alt="You need to choose two pieces!"
            class="pixelart text-wrapper"
            :style="characterPosition"
          >
          <img
            v-if=" errorMessage == 'Two Piece Need To Be Conjunctive'"
            src="../assets/twopiececonjunctive.png"
            alt="Two pieces need to be conjunctive!"
            class="pixelart text-wrapper"
            :style="characterPosition"
          >
          <img
            v-if=" errorMessage == 'Two Piece cant belong to the same player'"
            src="../assets/twopiecescantbelongtothesame.png"
            alt="Two Piece cant belong to the same player!"
            class="pixelart text-two-line-wrapper"
            :style="characterPosition"
          >
          <div class="character-wrapper" :style="[characterWalking, characterPosition]">
            <div
              class="character"
              v-bind:class="{          
          'animation__translate__south': (directionHeading == 'south' && walking == true),
          'animation__translate__east': (directionHeading == 'east' && walking == true),
          'animation__translate__north': (directionHeading == 'north' && walking == true),
          'animation__translate__west': (directionHeading == 'west' && walking == true)
          }"
            >
              <img
                src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/21542/DemoRpgCharacterShadow.png"
                alt="Shadow"
                class="character_shadow pixelart"
              >
              <img
                src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/21542/DemoRpgCharacter.png"
                alt="character"
                class="pixelart__stand pixelart"
                v-bind:class="{
            'animation__pixelart__stepping': walking == true,
            'pixelart__face__south':directionHeading == 'south',
            'pixelart__face__east':directionHeading == 'east',
            'pixelart__face__north':directionHeading == 'north',
            'pixelart__face__west':directionHeading == 'west'
            }"
              >
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    playerNum: Number
  },
  data: function() {
    return {
      playerList: [],
      playerNowIndex: 0,
      round: 1,
      winner: "",
      board: [],
      step: 1,
      directionChoose: "forward",
      directionHeading: "south",
      directionOrigin: "south",
      dice: 2,
      diceShow: 2,
      shaking: false,
      opening: false,
      walking: false,
      characterLeft: 160,
      characterTop: 150,
      textShow: false,
      position: [],
      grassPieces: [],
      grassPieceOnei: -1,
      grassPieceOnej: -1,
      grassPieceTwoi: -1,
      grassPieceTwoj: -1,
      routing: [],
      errorMessage: "",
      paymentNeedChecking: [],
      paymentChecked: [],
      playerPayTo: "",
      payment: 0
    };
  },
  computed: {
    playerNow() {
      return this.playerList[this.playerNowIndex].playerId;
    },
    characterWalking() {
      return {
        "--character-step": this.dice
      };
    },
    characterPosition() {
      return {
        "--character-top": this.characterTop + "px",
        "--character-left": this.characterLeft + "px"
      };
    },
    grassPiecesLength() {
      return this.grassPieces.length;
    },
    boardLength() {
      return this.board.length;
    },
    paymentNeedCheckingLength() {
      return this.paymentNeedChecking.length;
    },
    paymentCheckedLength() {
      return this.paymentChecked.length;
    }
  },
  mounted: function() {
    // Make playerList
    for (let i = 0; i < this.playerNum; i++) {
      this.playerList.push({
        playerId: "player" + (i + 1),
        coinNum: 30,
        grassNum: 15,
        blockNum: 0
      });
    }

    // Make board
    var line = [];
    for (let i = 0; i < 7; i++) {
      line.push("");
    }
    for (let i = 0; i < 7; i++) {
      this.board.push(line.slice());
    }

    this.position.push(3, 3);
  },

  methods: {
    // Step1 direction functions
    calcDirection() {
      switch (this.directionOrigin) {
        case "north":
          if (this.directionChoose == "left") {
            this.directionHeading = "west";
          }
          if (this.directionChoose == "right") {
            this.directionHeading = "east";
          }
          if (this.directionChoose == "forward") {
            this.directionHeading = "north";
          }
          break;
        case "south":
          if (this.directionChoose == "left") {
            this.directionHeading = "east";
          }
          if (this.directionChoose == "right") {
            this.directionHeading = "west";
          }
          if (this.directionChoose == "forward") {
            this.directionHeading = "south";
          }
          break;
        case "west":
          if (this.directionChoose == "left") {
            this.directionHeading = "south";
          }
          if (this.directionChoose == "right") {
            this.directionHeading = "north";
          }
          if (this.directionChoose == "forward") {
            this.directionHeading = "west";
          }
          break;
        case "east":
          if (this.directionChoose == "left") {
            this.directionHeading = "north";
          }
          if (this.directionChoose == "right") {
            this.directionHeading = "south";
          }
          if (this.directionChoose == "forward") {
            this.directionHeading = "east";
          }
          break;
        default:
          break;
      }
    },
    directionDecide() {
      this.directionOrigin = this.directionHeading;
      this.nextStepOfGame();
      this.directionChoose = "forward";
    },

    // Step2 dice functions
    shakeShaker() {
      this.shaking = true;
    },
    openShaker() {
      this.diceShow = Math.ceil(Math.random() * 6);
      this.dice = this.diceShow;
      this.opening = true;
      this.doneShake();
    },
    doneShake() {
      window.setTimeout(this.Step3OfGame, 1500 * this.diceShow + 600 * 3);
      window.setTimeout(
        this.resetDiceAnimation,
        1500 * this.diceShow + 600 * 3
      );
      window.setTimeout(this.calculatePayment, 1500 * this.diceShow + 600 * 3);
      window.setTimeout(this.emptyRouting, 1500 * this.diceShow + 600 * 3);

      this.calculateRouting();
      let isFirstZero = this.isFirstRouteZero();
      if (isFirstZero) {
        this.routing.splice(0, 1);
      }

      // let animation follow the routing
      let routingLength = this.routing.length;
      let timer = 0;
      this.runRoutine(0);
      if (routingLength >= 2) {
        timer = 1500 * this.routing[0].step + 600;
        window.setTimeout(this.runRoutine, timer, 1);
      }
      if (routingLength >= 3) {
        timer = timer + 1500 * this.routing[1].step + 600;
        window.setTimeout(this.runRoutine, timer, 2);
      }
      window.setTimeout(this.resetDirection, timer);
    },
    resetDirection() {
      this.directionOrigin = this.directionHeading;
    },

    // Functions in doneShake()
    calculateRouting() {
      let leftoverCells = 0;
      switch (this.directionHeading) {
        case "south":
          leftoverCells = this.boardLength - this.position[0] - 1;

          if (leftoverCells < this.dice) {
            this.pushSouthToRouting(leftoverCells);

            if (this.position[1] == this.boardLength - 1) {
              this.pushWestToRouting(this.dice - leftoverCells);
              return;
            }
            if (this.position[1] % 2 == 0) {
              this.pushEastToRouting(1);
              if (this.dice > leftoverCells + 1) {
                this.pushNorthToRouting(this.dice - leftoverCells - 1);
              }
              return;
            }
            if (this.position[1] % 2 == 1) {
              this.pushWestToRouting(1);
              if (this.dice > leftoverCells + 1) {
                this.pushNorthToRouting(this.dice - leftoverCells - 1);
              }
              return;
            }
          } else {
            this.pushSouthToRouting(this.dice);
            return;
          }
          break;

        case "east":
          leftoverCells = this.boardLength - this.position[1] - 1;

          if (leftoverCells < this.dice) {
            this.pushEastToRouting(leftoverCells);

            if (this.position[0] == this.boardLength - 1) {
              this.pushNorthToRouting(this.dice - leftoverCells);
              return;
            }
            if (this.position[0] % 2 == 0) {
              this.pushSouthToRouting(1);
              if (this.dice > leftoverCells + 1) {
                this.pushWestToRouting(this.dice - leftoverCells - 1);
              }
              return;
            }
            if (this.position[0] % 2 == 1) {
              this.pushNorthToRouting(1);
              if (this.dice > leftoverCells + 1) {
                this.pushWestToRouting(this.dice - leftoverCells - 1);
              }
              return;
            }
          } else {
            this.pushEastToRouting(this.dice);
            return;
          }
          break;

        case "north":
          leftoverCells = this.position[0];

          if (leftoverCells < this.dice) {
            this.pushNorthToRouting(leftoverCells);

            if (this.position[1] == 0) {
              this.pushEastToRouting(this.dice - leftoverCells);
              return;
            }
            if (this.position[1] % 2 == 1) {
              this.pushEastToRouting(1);
              if (this.dice > leftoverCells + 1) {
                this.pushSouthToRouting(this.dice - leftoverCells - 1);
              }
              return;
            }
            if (this.position[1] % 2 == 0) {
              this.pushWestToRouting(1);
              if (this.dice > leftoverCells + 1) {
                this.pushSouthToRouting(this.dice - leftoverCells - 1);
              }
              return;
            }
          } else {
            this.pushNorthToRouting(this.dice);
            return;
          }
          break;

        case "west":
          leftoverCells = this.position[1];

          if (leftoverCells < this.dice) {
            this.pushWestToRouting(leftoverCells);

            if (this.position[0] == 0) {
              this.pushSouthToRouting(this.dice - leftoverCells);
              return;
            }
            if (this.position[0] % 2 == 1) {
              this.pushSouthToRouting(1);
              if (this.dice > leftoverCells + 1) {
                this.pushEastToRouting(this.dice - leftoverCells - 1);
              }
              return;
            }
            if (this.position[0] % 2 == 0) {
              this.pushNorthToRouting(1);
              if (this.dice > leftoverCells + 1) {
                this.pushEastToRouting(this.dice - leftoverCells - 1);
              }
              return;
            }
          } else {
            this.pushWestToRouting(this.dice);
            return;
          }
          break;

        default:
          break;
      }
    },
    isFirstRouteZero() {
      let first = this.routing[0];
      if (first.step == 0) {
        return true;
      }
      return false;
    },
    runRoutine(i) {
      let path = this.routing[i];
      this.directionHeading = path.direction;
      this.dice = path.step;
      this.walking = true;
      // after walking the position need to be recalculate
      window.setTimeout(this.calculatePosition, 1500 * this.dice + 500);
      window.setTimeout(this.resetWalkAnimation, 1500 * this.dice + 500);
    },
    emptyRouting() {
      this.routing = [];
    },

    // Functions in calculateRouting()
    pushNorthToRouting(steps) {
      this.routing.push({
        direction: "north",
        step: steps
      });
    },
    pushSouthToRouting(steps) {
      this.routing.push({
        direction: "south",
        step: steps
      });
    },
    pushEastToRouting(steps) {
      this.routing.push({
        direction: "east",
        step: steps
      });
    },
    pushWestToRouting(steps) {
      this.routing.push({
        direction: "west",
        step: steps
      });
    },

    // Functions in runRoutine()
    calculatePosition() {
      switch (this.directionHeading) {
        case "south":
          this.characterTop = this.characterTop + this.dice * 50;
          break;
        case "east":
          this.characterLeft = this.characterLeft + this.dice * 50;
          break;
        case "north":
          this.characterTop = this.characterTop - this.dice * 50;
          break;
        case "west":
          this.characterLeft = this.characterLeft - this.dice * 50;
          break;

        default:
          break;
      }

      this.position.pop();
      this.position.pop();

      let topStart = -20;
      let leftStart = -7;
      let i = Math.round((this.characterTop - topStart) / 50);
      let j = Math.round((this.characterLeft - leftStart) / 50);
      this.position.push(i, j);
    },
    resetWalkAnimation() {
      this.walking = false;
    },

    // Step3 payment functions
    calculatePayment() {
      let playerBelonging = this.board[this.position[0]][this.position[1]];

      if (playerBelonging != "" && playerBelonging != this.playerNow) {
        // initiate the first step
        this.paymentNeedChecking.push([this.position[0], this.position[1]]);
        this.paymentNeedChecking = this.calculatePaymentHorizontal(
          playerBelonging
        ).slice();
        this.paymentNeedChecking.push([this.position[0], this.position[1]]);

        while (this.paymentNeedCheckingLength != 0) {
          this.shiftInChecked();
          this.paymentNeedChecking = this.calculatePaymentVertical(
            playerBelonging
          );
          this.removeOverlapInNeedChecking();
          if (this.paymentNeedCheckingLength == 0) {
            break;
          }

          this.shiftInChecked();
          this.paymentNeedChecking = this.calculatePaymentHorizontal(
            playerBelonging
          );
          this.removeOverlapInNeedChecking();
        }

        this.payment = this.paymentCheckedLength;
        this.playerPayTo = playerBelonging;
      } else {
        // player dont need to pay
        this.nextStepOfGame();
        this.payment = 0;
        this.playerPayTo = "";
        this.paymentChecked = [];
        this.paymentNeedChecking = [];
        this.step4Start();
      }
    },
    calculatePaymentVertical(playerBelonging) {
      let newNeedChecking = [];
      let len = this.paymentNeedCheckingLength;

      for (let index = 0; index < len; index++) {
        // For each piece, found all the pieces with the same column
        let rowId = this.paymentNeedChecking[index][0];
        let columnId = this.paymentNeedChecking[index][1];

        for (let i = 1; i <= rowId; i++) {
          if (this.board[rowId - i][columnId] == playerBelonging) {
            newNeedChecking.push([rowId - i, columnId]);
          } else {
            break;
          }
        }
        for (let i = 1; i < this.boardLength - rowId; i++) {
          if (this.board[rowId + i][columnId] == playerBelonging) {
            newNeedChecking.push([rowId + i, columnId]);
          } else {
            break;
          }
        }
      }

      return newNeedChecking;
    },
    calculatePaymentHorizontal(playerBelonging) {
      let newNeedChecking = [];
      let len = this.paymentNeedCheckingLength;

      for (let index = 0; index < len; index++) {
        // For each piece, found all the pieces with the same row
        let rowId = this.paymentNeedChecking[index][0];
        let columnId = this.paymentNeedChecking[index][1];

        for (let j = 1; j <= columnId; j++) {
          if (this.board[rowId][columnId - j] == playerBelonging) {
            newNeedChecking.push([rowId, columnId - j]);
          } else {
            break;
          }
        }
        for (let j = 1; j < this.boardLength - columnId; j++) {
          if (this.board[rowId][columnId + j] == playerBelonging) {
            newNeedChecking.push([rowId, columnId + j]);
          } else {
            break;
          }
        }
      }

      return newNeedChecking;
    },
    shiftInChecked() {
      let len = this.paymentNeedCheckingLength;
      for (let index = 0; index < len; index++) {
        let piece = this.paymentNeedChecking[index];
        this.paymentChecked.push(piece.slice());
      }
    },
    removeOverlapInNeedChecking() {
      let checkedlen = this.paymentCheckedLength;

      for (let checkedIndex = 0; checkedIndex < checkedlen; checkedIndex++) {
        let needRemoving = [];
        for (
          let needCheckingIndex = 0;
          needCheckingIndex < this.paymentNeedCheckingLength;
          needCheckingIndex++
        ) {
          if (
            JSON.stringify(this.paymentChecked[checkedIndex]) ==
            JSON.stringify(this.paymentNeedChecking[needCheckingIndex])
          ) {
            needRemoving.push(needCheckingIndex);
          }
        }

        let len = needRemoving.length;
        for (let index = 0; index < len; index++) {
          let removeIndex = needRemoving.pop();
          this.paymentNeedChecking.splice(removeIndex, 1);
        }
      }

      let initlen = this.paymentNeedCheckingLength;
      for (let i = 0; i < initlen; i++) {
        let needRemoving = [];
        let newlen = this.paymentNeedCheckingLength;
        if (i >= newlen) {
          break;
        }

        for (let j = i + 1; j < newlen; j++) {
          if (
            JSON.stringify(this.paymentNeedChecking[i]) ==
            JSON.stringify(this.paymentNeedChecking[j])
          ) {
            needRemoving.push(j);
          }
        }

        let len = needRemoving.length;
        for (let i = 0; i < len; i++) {
          let index = needRemoving.pop();
          this.paymentNeedChecking.splice(index, 1);
        }
      }
    },
    pay() {
      let coinBeforePay = this.playerList[this.playerNowIndex].coinNum;
      let playerGotPaidIndex = this.getPlayerIndex(this.playerPayTo);
      let playerBeforeGot = this.playerList[playerGotPaidIndex].coinNum;

      this.playerList[this.playerNowIndex]["coinNum"] = coinBeforePay - this.payment;
      this.playerList[playerGotPaidIndex]["coinNum"] = playerBeforeGot + this.payment;

      this.nextStepOfGame();
      this.payment = 0;
      this.playerPayTo = "";
      this.paymentChecked = [];
      this.paymentNeedChecking = [];

      this.step4Start();
    },
    getPlayerIndex(id) {
      for (let i = 0; i < this.playerNum; i++) {
        if (this.playerList[i].playerId == id) {
          return i;
        }
      }
    },

    // Step4 put grass functions
    step4Start() {
      if (this.playerList[this.playerNowIndex].grassNum == 0) {
        this.errorMessage = "Grasses are used up";
        return;
      } else {
        this.textShow = true;
      }
    },
    chooseCell(indexLine, indexCell) {
      if (this.isCellAvailble(indexLine, indexCell) && this.step == 4) {
        this.grassPieces.push([indexLine, indexCell]);

        if (this.grassPiecesLength == 1) {
          this.grassPieceOnei = indexLine;
          this.grassPieceOnej = indexCell;
        }

        if (this.grassPiecesLength == 2) {
          this.grassPieceTwoi = indexLine;
          this.grassPieceTwoj = indexCell;
        }
      }
    },
    isCellAvailble(indexLine, indexCell) {
      // Check if that cell is around the character
      if (
        indexLine < this.position[0] - 1 ||
        indexLine > this.position[0] + 1
      ) {
        return false;
      }

      if (
        indexCell < this.position[1] - 1 ||
        indexCell > this.position[1] + 1
      ) {
        return false;
      }

      if (indexLine == this.position[0] && indexCell == this.position[1]) {
        return false;
      }

      return true;
    },
    resetGrassChoose() {
      this.grassPieces = [];
      this.grassPieceOnei = -1;
      this.grassPieceOnej = -1;
      this.grassPieceTwoi = -1;
      this.grassPieceTwoj = -1;
    },
    placeGrass() {
      this.textShow = false;
      if (this.playerList[this.playerNowIndex].grassNum == 0) {
        this.errorMessage = "Grasses are used up";
        this.resetGrassChoose();
        return;
      }
      if (this.grassPiecesLength != 2) {
        this.errorMessage = "You need to choose two pieces";
        this.resetGrassChoose();
        return;
      }
      if (!this.isGrassPlaceRight()) {
        this.errorMessage = "Two Piece Need To Be Conjunctive";
        this.resetGrassChoose();
        return;
      }
      if (!this.isColorRight()) {
        this.errorMessage = "Two Piece cant belong to the same player";
        this.resetGrassChoose();
        return;
      }

      this.errorMessage = "";

      this.updateBlocks(this.board[this.grassPieceOnei][this.grassPieceOnej]);
      this.updateBlocks(this.board[this.grassPieceTwoi][this.grassPieceTwoj]);

      this.board[this.grassPieceOnei][this.grassPieceOnej] = this.playerNow;
      this.board[this.grassPieceTwoi][this.grassPieceTwoj] = this.playerNow;
      let newGrassNum = this.playerList[this.playerNowIndex].grassNum - 1;
      this.playerList[this.playerNowIndex]["grassNum"] = newGrassNum;

      this.resetGrassChoose();
      if (!this.isGameFinish()) {
        this.playerNowIndex = this.nextPlayer();
        this.step = 1;
      } else {
        this.step = 0;
      }
    },
    skipGrassChoose() {
      this.textShow = false;
      this.errorMessage = "";
      this.resetGrassChoose();
      if (!this.isGameFinish()) {
        this.playerNowIndex = this.nextPlayer();
        this.step = 1;
      } else {
        this.step = 0;
      }
    },

    // Functions in placeGrass()
    isGrassPlaceRight() {
      if (
        this.grassPieceOnei == this.grassPieceTwoi &&
        Math.abs(this.grassPieceOnej - this.grassPieceTwoj) == 1
      ) {
        return true;
      }
      if (
        this.grassPieceOnej == this.grassPieceTwoj &&
        Math.abs(this.grassPieceOnei - this.grassPieceTwoi) == 1
      ) {
        return true;
      }
      return false;
    },
    isColorRight() {
      if (
        this.board[this.grassPieceOnei][this.grassPieceOnej] == "" ||
        this.board[this.grassPieceTwoi][this.grassPieceTwoj] == ""
      ) {
        return true;
      }
      if (
        this.board[this.grassPieceOnei][this.grassPieceOnej] !=
        this.board[this.grassPieceTwoi][this.grassPieceTwoj]
      ) {
        return true;
      }
      return false;
    },
    updateBlocks(originalPlayer) {
      if (this.playerNow != originalPlayer && originalPlayer != "") {
        let thisplayer = this.playerList[this.getPlayerIndex(originalPlayer)];
        let blocks = thisplayer.blockNum;
        thisplayer["blockNum"] = blocks - 1;
        thisplayer = this.playerList[this.playerNowIndex];
        blocks = thisplayer.blockNum;
        thisplayer["blockNum"] = blocks + 1;

        return;
      }
      if (originalPlayer == "") {
        let thisplayer = this.playerList[this.playerNowIndex];
        let blocks = thisplayer.blockNum;
        thisplayer["blockNum"] = blocks + 1;
      }
    },
    nextPlayer() {
      let nextIndex = this.playerNowIndex + 1;
      if (nextIndex >= this.playerNum) {
        return 0;
      }
      return nextIndex;
    },

    // Check Winner
    isGameFinish() {
      if (this.playerNowIndex != this.playerNum - 1) {
        return false;
      }
      this.round = this.round + 1;
      for (let i = 0; i < this.playerNum; i++) {
        if (this.playerList[i].grassNum != 0) {
          return false;
        }
      }
      this.getWinner();
      return true;
    },
    getWinner() {
      let winnerIndex = -1;
      let winnerCoin = -1;
      let winnerBlock = -1;

      for (let i = 0; i < this.playerNum; i++) {
        if (this.playerList[i].coinNum > winnerCoin) {
          winnerIndex = i;
          winnerCoin = this.playerList[i].coinNum;
          winnerBlock = this.playerList[i].blockNum;
        }
        if (this.playerList[i].coinNum == winnerCoin) {
          console.log("equal");
          if (this.playerList[i].blockNum > winnerBlock) {
            console.log("replace");
            winnerIndex = i;
            winnerCoin = this.playerList[i].coinNum;
            winnerBlock = this.playerList[i].blockNum;
          }
        }
      }

      this.winner = this.playerList[winnerIndex].playerId;
    },

    // Other
    nextStepOfGame() {
      this.step = this.step + 1;
    },
    Step3OfGame() {
      this.step = 3;
    },
    resetDiceAnimation() {
      this.shaking = false;
      this.opening = false;
    }
  }
};
</script>

<style scoped>
/* CSS naming: BEM */
.competition {
  --pixel-size: 2;
  --pixel-shaker-size: 3;

  display: grid;
  grid-template-columns: 270px auto;
  grid-column-gap: 10px;
}
.game-round {
  margin-bottom: 20px;
  border: 1px solid black;
  border-radius: 3px;
  padding-top: 3px;
}

.playerList {
  margin-right: 0px;
}
td {
  width: 80px;
  height: 30px;
  border: 1px solid black;
  border-collapse: collapse;
  text-align: center;
}
.playList__playerNow {
  width: 30px;
  border: none;
}

.game__cell {
  width: 50px;
  height: 50px;
  border: 1px solid black;
  border-collapse: collapse;
  padding: 0px;
}
.player2-land_title {
  background-image: url("../assets/player2land.png");
  background-position-x: -12px;
  background-color: blue;
}
.player3-land_title {
  background-image: url("../assets/player3land.png");
  background-position-x: -20px;
  background-position-y: -20px;
  background-color: purple;
}
.player4-land_title {
  background-image: url("../assets/player4land.png");
  background-position-x: -5px;
  background-position-y: -20px;
  background-color: yellow;
}
.player1-land {
  background-image: url("../assets/player1land.png");
  background-color: green;
}
.player2-land {
  background-image: url("../assets/player2land.png");
  background-color: blue;
}
.player3-land {
  background-image: url("../assets/player3land.png");
  background-color: purple;
}
.player4-land {
  background-image: url("../assets/player4land.png");
  background-color: yellow;
}
.cell-chosen__player1 {
  width: 100%;
  height: 100%;
  border: 2px solid #00ff89;
}
.cell-chosen__player2 {
  width: 100%;
  height: 100%;
  border: 2px solid #00b9ff;
}
.cell-chosen__player3 {
  width: 100%;
  height: 100%;
  border: 2px solid #c65ee8;
}
.cell-chosen__player4 {
  width: 100%;
  height: 100%;
  border: 2px solid #ffd400;
}

.pixelart {
  image-rendering: pixelated;
}
button {
  display: block;
  font-family: "Neucha", sans-serif;
  font-size: 1em;
  margin: 10px auto;
  padding: 4px 10px 0 10px;
  border-radius: 3px;
}
.button-inline {
  display: inline-block;
  margin: 10px 10px;
}

/* dice and dice shaker */
.diceNshaker {
  position: relative;
}
.dice {
  width: 32px;
  height: 32px;
  position: absolute;
  left: 120px;
  top: 40px;
  z-index: -1;
}
.diceShaker {
  width: calc(32px * var(--pixel-shaker-size));
  height: calc(32px * var(--pixel-shaker-size));
}
.shakerShake {
  animation-name: shake;
  animation-duration: 0.5s;
  animation-iteration-count: infinite;
  animation-fill-mode: forwards;
}
@keyframes shake {
  0% {
    transform: translate(1px, 1px) rotate(0deg);
  }
  10% {
    transform: translate(-1px, -2px) rotate(-15deg);
  }
  20% {
    transform: translate(-3px, 0px) rotate(15deg);
  }
  30% {
    transform: translate(3px, 2px) rotate(0deg);
  }
  40% {
    transform: translate(1px, -1px) rotate(15deg);
  }
  50% {
    transform: translate(-1px, 2px) rotate(-15deg);
  }
  60% {
    transform: translate(-3px, 1px) rotate(0deg);
  }
  70% {
    transform: translate(3px, 1px) rotate(-15deg);
  }
  80% {
    transform: translate(-1px, -1px) rotate(15deg);
  }
  90% {
    transform: translate(1px, 2px) rotate(0deg);
  }
  100% {
    transform: translate(1px, -2px) rotate(-10deg);
  }
}
.shakerOpen {
  animation-name: open;
  animation-duration: 0.5s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
}
@keyframes open {
  0% {
    transform: translate(0px, 0px) rotate(0deg);
  }
  100% {
    transform: translate(40px, -40px) rotate(45deg);
  }
}

/* character */
.board-background-img {
  background-image: url("../assets/bgi-switch.png");
  width: 414px;
  height: 414px;
  margin-left: auto;
  margin-right: auto;
}
.board {
  position: relative;
  margin-top: 31px;
  margin-left: 31px;
  width: max-content;
  height: max-content;
}
.text-wrapper {
  display: block;
  position: absolute;
  top: calc(var(--character-top) - 25px);
  left: calc(var(--character-left) - 30px);
}
.text-two-line-wrapper {
  display: block;
  position: absolute;
  top: calc(var(--character-top) - 40px);
  left: calc(var(--character-left));
}
.character-wrapper {
  position: absolute;
  top: var(--character-top);
  left: var(--character-left);
}
.character {
  width: calc(16px * var(--pixel-size));
  height: calc(19px * var(--pixel-size));
  overflow: hidden;
  position: relative;
  text-align: justify;
}
.character_shadow {
  position: absolute;
  left: -16px;
  top: -25px;
  width: calc(32px * var(--pixel-size));
  height: calc(32px * var(--pixel-size));
}

/* CSS naming: functional CSS */
/* character animation */
.pixelart__face__south {
  top: calc((0px - 12px) * var(--pixel-size));
  left: calc(-8px * var(--pixel-size));
}
.pixelart__face__east {
  top: calc((-32px - 12px) * var(--pixel-size));
  left: calc(-8px * var(--pixel-size));
}
.pixelart__face__north {
  top: calc((-64px - 12px) * var(--pixel-size));
  left: calc(-8px * var(--pixel-size));
}
.pixelart__face__west {
  top: calc((-96px - 12px) * var(--pixel-size));
  left: calc(-8px * var(--pixel-size));
}
.pixelart__stand {
  width: calc(128px * var(--pixel-size));
  position: absolute;
}

.animation__pixelart__stepping {
  animation: characterstepping 0.5s steps(4, end);
  animation-iteration-count: calc(3 * var(--character-step) + 1);
  width: calc(128px * var(--pixel-size));
  position: absolute;
}
@keyframes characterstepping {
  from {
    transform: translate3d(0px, 0, 0);
  }
  to {
    transform: translate3d(-100%, 0, 0);
  }
}

/* animation name go* is to move on that direction several steps
    duration depend on how many steps */
.animation__translate__south {
  animation: goSouth linear forwards;
  animation-delay: 0.5s;
  animation-duration: calc(1.5s * var(--character-step));
}
.animation__translate__north {
  animation: goNorth linear forwards;
  animation-delay: 0.5s;
  animation-duration: calc(1.5s * var(--character-step));
}
.animation__translate__west {
  animation: goWest linear forwards;
  animation-delay: 0.5s;
  animation-duration: calc(1.5s * var(--character-step));
}
.animation__translate__east {
  animation: goEast linear forwards;
  animation-delay: 0.5s;
  animation-duration: calc(1.5s * var(--character-step));
}

@keyframes goSouth {
  from {
    transform: translate(0, 0);
  }
  to {
    transform: translate(0, calc(50px * var(--character-step)));
  }
}
@keyframes goNorth {
  from {
    transform: translate(0, 0);
  }
  to {
    transform: translate(0, calc(-50px * var(--character-step)));
  }
}
@keyframes goWest {
  from {
    transform: translate(0, 0);
  }
  to {
    transform: translate(calc(-50px * var(--character-step)), 0);
  }
}
@keyframes goEast {
  from {
    transform: translate(0, 0);
  }
  to {
    transform: translate(calc(50px * var(--character-step)), 0);
  }
}
</style>
