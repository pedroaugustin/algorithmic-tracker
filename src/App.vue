<template>
    <div class="appBackground">
        <div class="h-24">
        <h1 class="text-[50px] md:text-[100px] appTitle font-['8_Bit_Arcade'] text-white">Algorithmic Tracker</h1>
        </div>
        <div class="z-10">
        <nav class="text-white">
            <router-link class="text-[40px] text-white navText font-['8_Bit_Arcade']" to="/">home</router-link> |
            <router-link class="text-[40px] text-white navText font-['8_Bit_Arcade']" to="/about">about</router-link>
        </nav>
        <router-view />
        </div>
    </div>
</template>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
}

nav {
    padding: 30px;
}

nav a {
    color: #000000;
}

nav a.router-link-exact-active {
    color: #37d706;
}

@font-face {
    font-family: "8 Bit Hud";
    src: url("~@/assets/fonts/8-bit-hud.ttf");
}

@font-face {
    font-family: "Karma";
    src: url("~@/assets/fonts/karma suture.ttf");
}

@font-face {
    font-family: "8 Bit Arcade";
    src: url("~@/assets/fonts/8-bit Arcade In.ttf");
}

.appBackground {
background-image: url("~@/assets/bg.png");
}
</style>