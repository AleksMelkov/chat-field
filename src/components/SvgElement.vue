<template>
  <svg
    :width="`${iconWidth}px`"
    :height="`${iconHeight}px`"
  >
    <use :xlink:href="require('../assets/icons.svg')+'#i-'+iconName" />
  </svg>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';

@Component
export default class SvgElement extends Vue {
  @Prop({
    default: 20,
  }) readonly iconWidth!: number;

  @Prop({
    default: 20,
  }) readonly iconHeight!: number;

  @Prop({
    required: true,
  }) readonly iconName!: string;
}
</script>
