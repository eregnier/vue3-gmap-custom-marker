<template>
  <div :style="{opacity: opacity}">
    <slot/>
  </div>
</template>
<script>

export default {
  props: {
    marker: {
      type: Object,
      default: undefined
    },
    offsetX: {
      type: Number,
      default: 0
    },
    offsetY: {
      type: Number,
      default: 0
    },
    alignment: {
      type: String,
      default: "top"
    },
    zIndex: {
      type: Number,
      default: 50
    },
    cssPosition: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    onMap() {
      const map = this.map
      const self = this

      class Overlay extends this.maps.OverlayView {
        constructor(map) {
          super();
          this.setMap(map);
          this.draw = () => this.repaint();
          this.setPosition = () => this.repaint();
        }

        repaint() {
          const div = self.$el;
          const projection = this.getProjection();
          if (projection && div) {
            const posPixel = projection.fromLatLngToDivPixel(self.latLng);
            let x, y;
            switch (self.alignment) {
              case "top":
                x = posPixel.x - div.offsetWidth / 2;
                y = posPixel.y - div.offsetHeight;
                break;
              case "bottom":
                x = posPixel.x - div.offsetWidth / 2;
                y = posPixel.y;
                break;
              case "left":
                x = posPixel.x - div.offsetWidth;
                y = posPixel.y - div.offsetHeight / 2;
                break;
              case "right":
                x = posPixel.x;
                y = posPixel.y - div.offsetHeight / 2;
                break;
              case "center":
                x = posPixel.x - div.offsetWidth / 2;
                y = posPixel.y - div.offsetHeight / 2;
                break;
              case "topleft":
              case "lefttop":
                x = posPixel.x - div.offsetWidth;
                y = posPixel.y - div.offsetHeight;
                break;
              case "topright":
              case "righttop":
                x = posPixel.x;
                y = posPixel.y - div.offsetHeight;
                break;
              case "bottomleft":
              case "leftop":
                x = posPixel.x - div.offsetWidth;
                y = posPixel.y;
                break;
              case "bottomright":
              case "rightbottom":
                x = posPixel.x;
                y = posPixel.y;
                break;
              default:
                throw new Error("Invalid alignment type of custom marker!");
            }
            if (self.cssPosition) {
              div.style.transform = `translate(${x + self.offsetX}px, ${y + self.offsetY}px)`;
            } else {
              div.style.left = x + self.offsetX + "px";
              div.style.top = y + self.offsetY + "px";
            }
            div.style["z-index"] = self.zIndex;
          }
        }

        onAdd() {
          const div = self.$el;
          const panes = this.getPanes();
          div.style.position = "absolute";
          div.style.display = "inline-block";
          div.style.zIndex = self.zIndex;
          panes.overlayLayer.appendChild(div);
          panes.overlayMouseTarget.appendChild(div);
          this.getDraggable = () => false;
          this.getPosition = () => {
            console.log('on get position', self.maps, self.lat, self.lng)
            return new self.maps.LatLng(self.lat, self.lng);
          };
        }

        onRemove() {
          if (self.$el) {
            const ua = window.navigator.userAgent
            const msie = ua.indexOf("MSIE ")
            if (msie > 0 || !!ua.match(/Trident.*rv:11\./)) {
              self.$el.parentNode.removeChild(self.$el)
            } else {
              self.$el.remove();
            }
          }
        }
      }

      this.$overlay = new Overlay(map);
      setTimeout(() => {
        if (this.$overlay) {
          this.$overlay.repaint();
          this.opacity = 1;
        }
      }, 100);
    },
  },
  data() {
    return {
      opacity: 0.01
    };
  },
  watch: {
    maps() {
      this.onMap()
    },
    marker: {
      handler() {
        if (this.$overlay) {
          this.$overlay.setPosition()
        }
      },
      deep: true
    },
    zIndex() {
      if (this.$overlay) {
        this.$overlay.repaint()
      }
    }
  },
  computed: {
    maps() {
      return this.$parent.api
    },
    map() {
      return this.$parent.map
    },
    lat() {
      return parseFloat(
          isNaN(this.marker.lat) ? this.marker.latitude : this.marker.lat
      );
    },
    lng() {
      return parseFloat(
          isNaN(this.marker.lng) ? this.marker.longitude : this.marker.lng
      );
    },
    latLng() {
      if (this.marker instanceof this.maps.LatLng) {
        return this.marker;
      }
      return new this.maps.LatLng(this.lat, this.lng);
    }
  },
  unmounted() {
    if (this.$overlay) {
      this.$overlay.setMap(null);
      this.$overlay = undefined;
    }
  }
};
</script>