<template>
    <div class="position-relative">
      <!-- Start topbar + canvas wrapper -->
      <div class="d-flex flex-column fabric-canvas-wrapper">
        <!-- Start topbar-->
        <div class="pa-3 font-weight-bold text-white d-flex justify-space-between align-center" style="background-color:#071a50;">
          <!-- Start topbar left content -->
            <div class="d-flex justify-space-around align-center w-50">
              <div class="w-25 d-flex justify-center">LIVE VIEW</div>
              <v-select v-model="selectedValue" label="Canvas type" :items="['Raw images', '2D Unwrap']" density="compact" class="canvasType d-flex justify-space-between"></v-select>
            </div>
           <!-- End topbar left content -->
          <v-btn icon="mdi-arrow-expand"></v-btn>
          <!-- End topbar-->
        </div>
        <canvas id="c" style="border:1px solid rgb(220, 220, 220);"></canvas>
        <!-- End topbar + canvas wrapper -->
      </div>
    </div>
  </template>
  
  <script>
    import * as fabric from 'fabric'

    export default {
    data() {
      return {
        canvas: null,
        isFullScreen: false,
        selectedValue: "Raw images"
      };
    },
    computed: {
      getTagColor() {
        return (tag) => {
          return tag === "OK" ? "bg-green" : "bg-red";
        };
      }
    },
    methods: {
      resizeCanvas() {
        setTimeout(() => {
          const outerCanvasContainer = document.querySelector('.fabric-canvas-wrapper');
          const rect = outerCanvasContainer.getBoundingClientRect();
          
          let newWidth = rect.width +1
          let newHeight = window.innerHeight - 100;
          
          this.canvas.setDimensions({width: newWidth, height: newHeight});
          this.canvas.renderAll();
        }, 50);
      },
      handleZoom(opt) {
        var delta = opt.e.deltaY;
        var pointer = this.canvas.getPointer(opt.e);
        var zoom = this.canvas.getZoom();
        zoom = zoom - delta / 200;
        if (zoom > 20) zoom = 20;
        if (zoom < 1) zoom = 1;
        this.canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
        opt.e.preventDefault();
        opt.e.stopPropagation();
      }
    },
    mounted() {
      let interval = setInterval(() => {
        if (window.fabric) {
          clearInterval(interval);
        this.canvas = new fabric.Canvas('c', {
          //backgroundColor: 'white',
        });
          this.resizeCanvas();
          window.addEventListener('resize', this.resizeCanvas);
  
          this.canvas.on('mouse:wheel', this.handleZoom);
  
          this.$socket.on('msg-input:' + this.id, (msg) => {
            // Elimina solo las imágenes, no las líneas de la cuadrícula
            this.canvas.getObjects().forEach(obj => {
              if (obj.type === 'image') {
                this.canvas.remove(obj);
              }
            });
  
            fabric.Image.fromURL(msg.payload.imageData, (img) => {
              this.canvas.add(img);
              this.canvas.viewportCenterObject(img);
              this.canvas.renderAll();
            });
          });
        }
      }, 100);
    },
    unmounted() {
      if (this.canvas) {
        this.canvas.dispose();
        this.canvas = null;
      }
      window.removeEventListener('resize', this.resizeCanvas);
    }
  }
  </script>
  
  <style>
  #c {
    background-size: 40px 40px;
    background-repeat: repeat;
    background-image: 
      linear-gradient(to right, #ddd 1px, transparent 1px), 
      linear-gradient(to bottom, #ddd 1px, transparent 1px);
  }
  
  .canvasType.v-select div.v-input__control {
      width:50%;
  }
  
  </style>