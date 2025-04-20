<template>
<section>
    <div>
        <img src="/public/pedro.jpeg" alt="Pedro Miguel">
        <ul>
            <li><a href="https://github.com/Pedro-Miguel20"><i class="fa-brands fa-github"></i></a></li>
            <li><a href="https://www.linkedin.com/in/pedro-miguel-aguiar-09544b238"><i class="fa-brands fa-linkedin"></i></a></li>
        </ul>
    </div>
    <div>
        <h2>
            About me
        </h2>
        <p>Hello, I am Pedro, I'm a college stundent whose focus is learning about programming, and this project is the representation of this focus, for this is my first app learning Vue, despite it has a lot of problems I hope you all like it</p>
        <div class="footermenu">
            <a href="/" class="balloon-button">
                return
            </a>
            <a href="" class="balloon-button">
                My Portfolio!
            </a>
        </div>
    </div>
</section>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Patrick+Hand&display=swap');


.footermenu{
    position: relative;
    top: 10px;
}
p{
    color: white;
    font-size: 24px;
}
ul{
    display: flex;
    list-style-type: none;
    justify-content: center;
    padding: 0;
    gap: 30px;
    font-size: 50px;
    margin: 20px;
}

section{
    display: flex;
    gap: 40px;
    padding: 0px 80px;
    align-items: center;
    height: 100vh;
}

img{
    width: 300px;
    border-radius: 50%;
    border: 2px solid #00f5d4;
    transition: .5s all;
    cursor: pointer;
}

h2{
    -webkit-text-stroke-color:#00f5d4 !important;
}

img:hover {
    transform: scale(1.05);
    box-shadow: 0 0 0 5px #4caf5030;
}

i:hover{
    color: #00f5d4;
    transition: 0.5s all;
}

.balloon-button {
    margin: 20px;
      background: linear-gradient(to top, #ff009d, #ff9999);
      border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
      width: 120px;
      height: 150px;
      position: relative;
      color: white;
      font-family: "Patrick Hand", cursive;
      font-weight: bold;
      font-size:26px;
      cursor: pointer;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
      padding:20px;
    }

    .balloon-button:hover {
        transform: scale(1.1);
    }

    .balloon-button::after {
      content: '';
      position: absolute;
      bottom: -15px;
      left: 50%;
      transform: translateX(-50%);
      width: 20px;
      height: 20px;
      background: #ff009d;
      clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
    }
</style>
