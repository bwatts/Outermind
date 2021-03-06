<template>
  <v-sheet
    :class="cardClass"
    :style="cardStyle"
    :elevation="Math.min(24, card.elevation)"
    tile
    @mouseenter="mousehover"
    @mousemove="mousehover"
    @mouseleave="mouseleave"
    @mousedown.prevent.stop="mousedown">

    <app-content :cardId="card.id" />

    <div v-if="arranging" class="overlay" :style="overlayStyle" @mousedown.prevent.stop="overlayMousedown">
      <div v-if="inspecting" class="top-corners" />
      <div v-if="inspecting" class="bottom-corners" />
    </div>
  </v-sheet>
</template>

<script>
  import AppContent from "./AppContent";

  export default {
    components: { AppContent },
    props: ["card"],
    data() {
      return {
        ctrl: false,
        shift: false,
        hover: null,
        drag: null
      }
    },
    computed: {
      arranging() {
        return this.ctrl && this.shift;
      },
      inspecting() {
        return this.ctrl && this.shift && this.hover;
      },
      cardClass() {
        return ["card", {
          inspecting: this.inspecting,
          dragging: !!this.drag
        }];
      },
      cardStyle() {
        let { rows, columns, row, column, elevation } = this.card;

        return {
          gridArea: `${row} / ${column} / span ${rows} / span ${columns}`,
          zIndex: elevation
        };
      },
      overlayStyle() {
        if(!this.hover || !this.hover.resize) {
          return;
        }

        let { north, south, west, east } = this.hover;

        let directions = [north, south, west, east];
        let symbols = ["n", "s", "w", "e"];
        let cursor = "";

        for(let i = 0; i < directions.length; i++) {
          if(directions[i]) {
            cursor += symbols[i];
          }
        }

        return !cursor ? null : { cursor: `${cursor}-resize` };
      }
    },
    mounted() {
      document.addEventListener("keydown", this.keychange);
      document.addEventListener("keyup", this.keychange);
    },
    beforeDestroy() {
      document.removeEventListener("keydown", this.keychange);
      document.removeEventListener("keyup", this.keychange);
    },
    methods: {
      keychange(e) {
        this.ctrl = e.ctrlKey;
        this.shift = e.shiftKey;
      },
      mousehover(e) {
        if(this.drag) {
          return;
        }

        let bounds = this.$el.getBoundingClientRect();

        let top = e.clientY - bounds.top;
        let left = e.clientX - bounds.left;
        let width = bounds.right - bounds.left;
        let height = bounds.bottom - bounds.top;

        let north = top < 20;
        let south = top > height - 20;
        let west = left < 20;
        let east = left > width - 20;

        this.hover = {
          north,
          south,
          west,
          east,
          resize: north || south || west || east
        };
      },
      mouseleave() {
        if(!this.drag) {
          this.hover = null;
        }
      },
      mousedown() {
        this.$emit("selectCard", { card: this.card });
      },
      overlayMousedown(e) {
        document.addEventListener("mousemove", this.mousemove);
        document.addEventListener("mouseup", this.mouseup);

        this.$emit("selectCard", { card: this.card });

        let bounds = this.$el.getBoundingClientRect();

        this.drag = {
          offsetX: e.clientX - bounds.x,
          offsetY: e.clientY - bounds.y
        };
      },
      mousemove(e) {
        let { card, hover, drag } = this;

        if(hover && hover.resize) {
          this.$emit("resizeCard", {
            card,
            hover,
            x: e.clientX,
            y: e.clientY
          });
        }
        else {
          this.$emit("dragCard", {
            card,
            x: e.clientX - drag.offsetX,
            y: e.clientY - drag.offsetY
          });
        }
      },
      mouseup() {
        document.removeEventListener("mousemove", this.mousemove);
        document.removeEventListener("mouseup", this.mouseup);

        this.drag = null;
      }
    }
  };
</script>

<style scoped>
  .card {
    margin: 1px 0 0 1px;
    cursor: default;
  }

  .card.inspecting {
    cursor: grab;
  }

  .card.dragging {
    cursor: grabbing;
  }

  .overlay {
    position: absolute;
    top: -2px;
    left: -2px;
    right: -2px;
    bottom: -2px;
    border: solid 2px #D1C4E9;
    margin: 1px;
  }

  .top-corners:before,
  .top-corners:after,
  .bottom-corners:before,
  .bottom-corners:after {
    position: absolute;
    width: 21px;
    height: 21px;
    content: '';
  }

  .top-corners:before {
    top: -2px;
    left: -2px;
    border-left: 2px solid #673AB7;
    border-top: 2px solid #673AB7;
  }

  .top-corners:after {
    top: -2px;
    right: -2px;
    border-right: 2px solid #673AB7;
    border-top: 2px solid #673AB7;
  }

  .bottom-corners:before {
    bottom: -2px;
    left: -2px;
    border-left: 2px solid #673AB7;
    border-bottom: 2px solid #673AB7;
  }

  .bottom-corners:after {
    bottom: -2px;
    right: -2px;
    border-right: 2px solid #673AB7;
    border-bottom: 2px solid #673AB7;
  }
</style>