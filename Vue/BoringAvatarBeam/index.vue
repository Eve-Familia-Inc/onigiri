<template>
  <div id="content"></div>
</template>

<script>
  import { getNumber, getUnit, getBoolean, getRandomColor, getContrast } from './utilities';
  class Random {
    constructor(seed = 88675123) {
      this.x = 123456789;
      this.y = 362436069;
      this.z = 521288629;
      this.w = seed;
    }
    // XorShift
    next() {
      let t;

      t = this.x ^ (this.x << 11);
      this.x = this.y; this.y = this.z; this.z = this.w;
      return this.w = (this.w ^ (this.w >>> 19)) ^ (t ^ (t >>> 8));
    }
    // min以上max以下の乱数を生成する
    nextInt(min, max) {
      const r = Math.abs(this.next());
      return min + (r % (max + 1 - min));
    }
  }
  function rgb2hex(rgb) {
    return "#" + rgb.map(function (value) {
      return ("0" + value.toString(16)).slice(-2);
    }).join("");
  }
  class BoringAvatarBeam {
    constructor(size, name, square) {
      this.SIZE = 36;
      this.size = size;
      this.name = name;
      this.colors = [];
      this.square = square == "true";
      this.data = this.generateData(this.name, this.colors);
    }
    generateData(name, colors) {
      const numFromName = getNumber(name);
      const random = new Random(numFromName);
      for (let i = 0; i < 5; i++) {
        let col = [];
        for (let j = 0; j < 3; j++) {
          const value = random.nextInt(0, 255);
          col.push(value);
        }
        this.colors.push(rgb2hex(col));
      }

      const range = colors && colors.length;
      const wrapperColor = getRandomColor(numFromName, colors, range);
      const preTranslateX = getUnit(numFromName, 10, 1);
      const wrapperTranslateX = preTranslateX < 5 ? preTranslateX + this.SIZE / 9 : preTranslateX;
      const preTranslateY = getUnit(numFromName, 10, 2);
      const wrapperTranslateY = preTranslateY < 5 ? preTranslateY + this.SIZE / 9 : preTranslateY;

      const data = {
        wrapperColor: wrapperColor,
        faceColor: getContrast(wrapperColor),
        backgroundColor: getRandomColor(numFromName + 13, colors, range),
        wrapperTranslateX: wrapperTranslateX,
        wrapperTranslateY: wrapperTranslateY,
        wrapperRotate: getUnit(numFromName, 360),
        wrapperScale: 1 + getUnit(numFromName, this.SIZE / 12) / 10,
        isMouthOpen: getBoolean(numFromName, 2),
        isCircle: getBoolean(numFromName, 1),
        eyeSpread: getUnit(numFromName, 5),
        mouthSpread: getUnit(numFromName, 3),
        faceRotate: getUnit(numFromName, 10, 3),
        faceTranslateX:
          wrapperTranslateX > this.SIZE / 6 ? wrapperTranslateX / 2 : getUnit(numFromName, 8, 1),
        faceTranslateY:
          wrapperTranslateY > this.SIZE / 6 ? wrapperTranslateY / 2 : getUnit(numFromName, 7, 2),
      };

      return data;
    }


    generateSVG(s) {

      let SVG = `<svg viewBox="0 0 ${this.SIZE} ${this.SIZE}" fill="none" xmlns="http://www.w3.org/2000/svg" width=${this.size} height=${this.size}>` +
        `<mask id="mask__beam" maskUnits="userSpaceOnUse" x=0 y=0 width=${this.SIZE} height=${this.SIZE}>` +
        `<rect width=${this.SIZE} height=${this.SIZE} rx=${this.square ? 0 : this.SIZE * 2} fill="white" />` +
        `</mask>` +
        `<g mask="url(#mask__beam)">` +
        `<rect width=${this.SIZE} height=${this.SIZE} fill=${this.data.backgroundColor} />` +
        `<rect x="0" y="0" width=${this.SIZE} height=${this.SIZE} transform="translate(${this.data.wrapperTranslateX} ${this.data.wrapperTranslateY}) rotate(${this.data.wrapperRotate} ${this.SIZE / 2} ${this.SIZE / 2}) scale(${this.data.wrapperScale})" fill=${this.data.wrapperColor} rx=${this.data.isCircle ? this.SIZE : this.SIZE / 6} />` +
        `<g transform="translate(${this.data.faceTranslateX} ${this.data.faceTranslateY}) rotate(${this.data.faceRotate} ${this.SIZE / 2} ${this.SIZE / 2})">` +
        (this.data.isMouthOpen ? `<path d='M15 ${19 + this.data.mouthSpread} c2 1 4 1 6 0' stroke=${this.data.faceColor} fill="none" strokeLinecap="round" />` : `<path d="M13,${19 + this.data.mouthSpread} a1,0.75 0 0,0 10,0" fill=${this.data.faceColor} />`) +
        `<rect x=${14 - this.data.eyeSpread} y=14 width=1.5 height=2 rx=1 stroke="none" fill=${this.data.faceColor} />` +
        `<rect x=${20 + this.data.eyeSpread} y=14 width=1.5 height=2 rx=1 stroke="none" fill=${this.data.faceColor} />` +
        `</g>` +
        `</g>` +
        `</svg>`;
      let div = document.getElementById(s);
      div.innerHTML = SVG;
    }
  }
  export default {
    name: 'BoringAvatarBeam',
    props: ['size', 'name', 'square'],

    mounted() {
      new BoringAvatarBeam(this.size, this.name, this.square).generateSVG("content");
    },
    watch: {
      name: function (newName, oldName) {
        new BoringAvatarBeam(this.size, newName, this.square).generateSVG("content");
      }
    }
  }
</script>