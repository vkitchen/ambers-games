<template>
  <div class="introduction-page">
    <h1>Games</h1>
    <div class="page-notice">
      * If you want to play online, after clicking the play button,
      send the URL which including room number to the other player to invite the other player to enter the room.
      <br>While the other player enters the URL, he/she will automatically enter the room.
      <br>* If you want to play on the same device, choose to play locally.
      <br>
      <br>* Click to see
      <RouterLink to="/notice" class="link__none-style">Changelog</RouterLink>
    </div>
    <ul>
      <li>
        <span class="fa fa-rotate-right icon"></span>
        <RouterLink to="/RotatingPuzzle/Introduction" class="link__none-style">Rotating puzzle (1 player)</RouterLink>
      </li>
      <li>
        <font-awesome-icon :icon="['fas', 'plane']" class="icon__airplane icon"/>
        <RouterLink to="/FindAirplane/Introduction" class="link__none-style">Seek for Airplane's Head (2 players)</RouterLink>
      </li>
      <!-- <li>
        <span class="fa fa-hand-scissors-o icon__scissor icon"></span>
        <router-link
          :to="{name: 'RockPaperScissorsIntroduction'}"
          class="link__none-style"
        >Rock Paper Scissors</router-link>
      </li>-->
      <li>
        <span class="fa fa-cube icon"></span>
        <RouterLink to="/TicTacToeBox/Introduction" class="link__none-style">Tic Tac Toe Box version (2 players)</RouterLink>
      </li>
      <li>
        <span class="icon">
          <img src="../assets/flower.png">
        </span>
        <RouterLink to="/Monopoly/Introduction" class="link__none-style">Monopoly (2-4 players)</RouterLink>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.page-notice {
  border: 2px dotted transparent;
  border-image: linear-gradient(to top, #f9a507, #00b176);
  border-image-slice: 10;
  padding: 10px;
}
ul {
  font-size: 1.25em;
}
li {
  line-height: 35px;
}
li:hover {
  text-decoration: underline;
}
.icon {
  display: inline-block;
  width: 30px;
  text-align: center;
}
</style>
