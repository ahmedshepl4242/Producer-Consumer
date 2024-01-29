<template></template>

<script>
export default {
  props: {
    layer: Object,
  },

  methods: {
    createObject(attributes) {
      var rect = new Konva.Rect({
        x: attributes.x,
        y: attributes.y,
        width: attributes.width,
        height: attributes.height,
        stroke:'black',
        strokeWidth: 2,
        id: attributes.id,
        draggable: true,
        rotation: attributes.rotation,
        scaleX: attributes.scaleX,
        scaleY: attributes.scaleY,
        fill: attributes.fill,
        name: 'rectangle'
      });

      this.layer.add(rect);
    },
  },
};
</script>

