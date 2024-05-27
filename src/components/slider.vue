<template>
    <div class="range-slider">
      <div class="slider-values">       
      </div>
      <div class="slider-track" @click="onTrackClick">
        <div class="slider-thumb" :style="{ left: `${leftThumb}%` }" @mousedown="startMove($event, 'min')">
          <span class="thumb-value">{{ minValue }}</span>
        </div>
        <div class="slider-thumb" :style="{ left: `${rightThumb}%` }" @mousedown="startMove($event, 'max')">
          <span class="thumb-value">{{ maxValue }}</span>
        </div>
        <div class="slider-range" :style="{ left: `${leftThumb}%`, width: `${rightThumb - leftThumb}%` }"></div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'RangeSlider',
    props: {
      min: {
        type: Number,
        default: 1
      },
      max: {
        type: Number,
        default: 151
      },
      value: {
        type: Array,
        default: () => [1, 151]
      }
    },
    data() {
      return {
        minValue: this.value[0],
        maxValue: this.value[1],
        leftThumb: (this.value[0] - this.min) / (this.max - this.min) * 100,
        rightThumb: (this.value[1] - this.min) / (this.max - this.min) * 100,
        activeThumb: null
      };
    },
    methods: {
      emitMinValue() {
        this.$emit('update:min', this.minValue);
      },
      emitMaxValue() {
        this.$emit('update:max', this.maxValue);
      },
      startMove(event, thumb) {
        this.activeThumb = thumb;
        document.addEventListener('mousemove', this.moveThumb);
        document.addEventListener('mouseup', this.stopMove);
      },
      moveThumb(event) {
        const sliderRect = this.$el.getBoundingClientRect();
        const dx = event.clientX - sliderRect.left;
        const newPosition = (dx / sliderRect.width) * 100;
        
        if (this.activeThumb === 'min' && newPosition < this.rightThumb) {
          this.leftThumb = Math.max(0, newPosition);
          this.minValue = Math.round(this.min + (this.max - this.min) * (this.leftThumb / 100));
          this.emitMinValue();
        } else if (this.activeThumb === 'max' && newPosition > this.leftThumb) {
          this.rightThumb = Math.min(100, newPosition);
          this.maxValue = Math.round(this.min + (this.max - this.min) * (this.rightThumb / 100));
          this.emitMaxValue();
        }
        
        
      },
      stopMove() {
        document.removeEventListener('mousemove', this.moveThumb);
        document.removeEventListener('mouseup', this.stopMove);
      },
      onTrackClick(event) {
        const sliderRect = this.$el.getBoundingClientRect();
        const dx = event.clientX - sliderRect.left;
        const clickPosition = (dx / sliderRect.width) * 100;
  
        const middle = (this.leftThumb + this.rightThumb) / 2;
        if (clickPosition < middle) {
          this.leftThumb = Math.max(0, clickPosition);
          this.minValue = Math.round(this.min + (this.max - this.min) * (this.leftThumb / 100));
          this.emitMinValue();
        } else {
          this.rightThumb = Math.min(100, clickPosition);
          this.maxValue = Math.round(this.min + (this.max - this.min) * (this.rightThumb / 100));
          this.emitMaxValue();
        }        
      },      
    }
  }
  </script>
  
  <style scoped>

  
  .range-slider {
    position: relative;
    width: 60%;
    height: 40px;
    margin-left: 20%;
  }
  
  .slider-values {
    display: flex;
    justify-content: space-between;
    padding: 0 10px;
  }
  
  .slider-track {
    background-color: #ddd;
    height: 4px;
    position: relative;
    top: 18px;
    width: 100%;
  }
  
  .slider-thumb {
    position: absolute;
    height: 20px;
    width: 20px;
    background-color: orangered;
    border-radius: 50%;
    cursor: pointer;
    margin-left: -10px;
    margin-top: -8px;
    z-index: 5;
  }
  
  .thumb-value {
    position: absolute;
    top: -30px; 
    left: 50%;
    transform: translateX(-50%);
    color: white;
  }
  
  .slider-range {
    position: absolute;
    height: 4px;
    background-color: red;
    top: 0;
    
  }
  </style>
  