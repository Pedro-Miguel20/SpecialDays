<template><div class="title"><h2>Special</h2><h3>Days</h3></div></template>

<style>
    @font-face {
        font-family: 'cookie';
        src: url('../Fonts/BakeryHomeFreeTrial.ttf');
    }
    @font-face {
        font-family: 'birthday';
        src: url('../Fonts/HappyBirthdayLetterCandles-Regular.ttf');
    }

    .title{
        position:absolute;
        display: flex;
        flex-direction: row-reverse;
        padding: 20px 20px;
        right: 0;
        z-index: 100;
        top: -60px;
    }

    h2, h3{
        font-family: 'cookie';
        font-size: 80px;
        color: white;
        margin: 0;
        -webkit-text-stroke-width: 2px; /* largura da borda */
    -webkit-text-stroke-color: #e960b5; /* cor da borda */
    background: url('download.jpg') no-repeat center center;
      background-size: cover;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    cursor: default;
    user-select: none;
        transform: rotate(90);
        transition: all 0.5s;
        filter: drop-shadow(2px 7px #c24e95);
    }

    h2:hover, h3:hover{
        transform: scale(1.1);

      transition: all 0.5s;
    }
    h3{
        position: fixed;
        top:25px;
        filter: drop-shadow(2px 7px #4ef3d3);
        -webkit-text-stroke-color: #4cd8bc;
    }

    /*{
        position: absolute;
        font-family: 'birthday';
        font-size: 80px;
        left: 50px;
        color: white;
        text-shadow: 2px 5px 5px #e960b5;
    }*/
</style>