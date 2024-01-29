<template></template>

<script>
export default {
  props: {
    layer: Object,
  },

  methods: {
    createObject(attributes) {
      var circ = new Konva.Circle({
        x: attributes.x,
        y: attributes.y,
        radius: attributes.radius,
        stroke:'black',
        strokeWidth: 2,
        id: attributes.id,
        draggable: true,
        rotation: attributes.rotation,
        scaleX: attributes.scaleX,
        scaleY: attributes.scaleY,
        fill: attributes.fill,
        name: 'circle'
      });

      this.layer.add(circ);
    },
  },
};
</script>

