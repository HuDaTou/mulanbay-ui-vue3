<template>
 <header>
    <a href="#" class="logo">Logo</a>
  </header>
  <section>
    <img src="./images/stars.png" id="stars">
    <img src="./images/moon.png" id="moon">
    <img src="./images/mountains_behind.png" id="mountains_behind">
    <h2 id="text">Moon Light</h2>
    <a href="#sec" id="btn">Explore</a>
    <img src="./images/mountains_front.png" id="mountains_front">
  </section>
  <div class="sec" id="sec">
    <h2>首页</h2>
    <p>P-life是用于管理个人消费、锻炼、音乐、阅读、健康、饮食、人生经历等各个衣食住行信息的系统，由统计、计划模块利用调度系统来统计分析执行情况。 并通过积分和评分体系来综合评估个人的总体状态，同时采用机器学习算法对数据进行预测。</p>
  </div>
</template>

<script setup name="Index">
import "./style.css"
let stars = document.getElementById('stars')
    let moon = document.getElementById('moon')
    let mountains_behind = document.getElementById('mountains_behind')
    let text = document.getElementById('text')
    let btn = document.getElementById('btn')
    let mountains_front = document.getElementById('mountains_front')
    let header = document.querySelector('header')

    window.addEventListener('scroll', function(){
      let value = window.scrollY;
      stars.style.left = value * 0.25 + 'px'
      moon.style.top = value * 1.05 + 'px'
      mountains_behind.style.top = value * 0.5 + 'px'
      mountains_front.style.top = value * 0 + 'px'
      text.style.marginRight = value * 4 + 'px'
      text.style.marginTop = value * 1.5 + 'px'
      btn.style.marginTop = value * 1.5 + 'px'
      header.style.top = value * 0.5 + 'px'
    })
</script>
