<template>
  <BaseCard class="ReportsBarGraph">
    <p class="ReportsBarGraph_Title">CLOCKED HOURS</p>
    <canvas
      ref="barGraph"
      class="ReportsBarGraph_Content"
      :width="`${contentWidth}px`"
      :height="`${contentHeight}px`"
    ></canvas>
  </BaseCard>
</template>

<script lang="ts">
import {
  forEach,
  map,
  max,
} from 'lodash';
import moment, { Moment } from 'moment';
import {
  Component,
  Prop,
  Ref,
  Vue,
  Watch,
} from 'vue-property-decorator';
import Formatter from '@/utils/formatter';
import ReportManager from '@/store/modules/ReportManager';
import BaseCard from '~/atoms/BaseCard.vue';

const loadingMaxScale = 10;
const dayOfWeek: string[] = ['S', 'M', 'T', 'W', 'T', 'F', 'S'];
const itemWidth: number = 30;
const margin: number = 12;
const graphItemMaxHeight: number = 152;
const graphMaxHeight: number = 192;
const graphScaleHeight: number = 40;

@Component({
  components: {
    BaseCard,
  },
})
export default class ReportsBarGraph extends Vue {
  @Prop({ default: false }) isLoading?: boolean;

  @Ref('barGraph') barGraphContent!: HTMLCanvasElement;

  private ctx!: CanvasRenderingContext2D | null;

  private maxScale: number = 0;

  private contentWidth: number = 0;

  private contentHeight: number = 0;

  private data: {[date: string]: number} = ReportManager.reportState.barGraph;

  mounted() {
    this.contentWidth = this.$el.clientWidth * 2;
    this.contentHeight = (graphMaxHeight + graphScaleHeight) * 2;

    const canvas = this.barGraphContent;
    this.ctx = canvas!.getContext('2d');
  }

  updated() {
    this.ctx!.scale(2, 2);
    if (this.isLoading) {
      this.drawVerticalScale(loadingMaxScale);
      this.drawLodingHorizontalScale();
    } else {
      this.refleshContent();
      this.drawGraph();
    }
  }

  @Watch('isLoading')
  private onChangeLoadingStatus(): void {
    if (this.isLoading) {
      this.drawVerticalScale(loadingMaxScale);
      this.drawLodingHorizontalScale();
    } else {
      this.refleshContent();
      this.drawGraph();
    }
  }

  private drawGraph(): void {
    this.maxScale = this.getMaxScale();
    this.drawVerticalScale(this.maxScale);
    this.drawHorizontalScale();
    this.drawGraphItem();
  }

  private drawVerticalScale(maxHour: number): void {
    if (typeof this.ctx === 'undefined') return;

    const fontWidth: number = 24;
    const scales = ReportsBarGraph.getVerticalScale(maxHour);
    forEach(scales, (scale: { [key: string]: number }) => {
      this.ctx!.fillStyle = '#C4C4C6';
      this.ctx!.font = '12px Helvetica Neue';
      this.ctx!.fillText(`${scale.hour} h`, this.contentWidth / 2 - (margin + fontWidth), scale.top - 8, fontWidth);
      this.drawLine(0, scale.top, this.contentWidth, scale.top, '#E5E5EA', 1);
    });
  }

  private drawLodingHorizontalScale(): void {
    this.ctx!.fillStyle = '#C4C4C6';
    this.ctx!.font = '13px Helvetica Neue';
    this.ctx!.fillText('1/4', margin, graphMaxHeight + 20);
    this.ctx!.fillText('30/4', this.contentWidth / 2 - 80, graphMaxHeight + 20);
  }

  private drawHorizontalScale(): void {
    if (typeof this.ctx === 'undefined') return;
    this.ctx!.beginPath();

    let moveLeft: number = margin;
    forEach(this.data, (count: number, date: string) => {
      const momentDate: Moment = moment(date);
      const weekday = dayOfWeek[momentDate.weekday()];

      this.ctx!.fillStyle = '#C4C4C6';
      this.ctx!.font = '300 13px Helvetica Neue';
      this.ctx!.fillText(weekday, moveLeft + margin, graphMaxHeight + 20);
      this.ctx!.font = '300 12px Helvetica Neue';
      this.ctx!.fillText(momentDate.format('MM/DD'), moveLeft, graphMaxHeight + 36);

      moveLeft += itemWidth + margin;
    });
  }

  private drawGraphItem(): void {
    if (typeof this.ctx === 'undefined') return;
    this.ctx!.beginPath();

    let moveLeft: number = margin;
    forEach(this.data, (count: number, date: string) => {
      const itemHeight = this.getGraphItemHeight(count);
      if (itemHeight === 0) {
        const top = graphMaxHeight - 1;
        this.drawLine(moveLeft, top, moveLeft + itemWidth, top);
      } else {
        this.ctx!.fillStyle = '#47C3FC';
        this.ctx!.fillRect(moveLeft, graphMaxHeight, itemWidth, -itemHeight);
      }

      moveLeft += itemWidth + margin;
    });
  }

  private drawLine(
    startX: number,
    startY: number,
    endX: number,
    endY: number,
    color: string = '#000',
    weight: number = 1,
  ): void {
    this.ctx!.strokeStyle = color;
    this.ctx!.lineWidth = weight;
    this.ctx!.moveTo(startX, startY);
    this.ctx!.lineTo(endX, endY);
    this.ctx!.stroke();
  }

  private getMaxScale(): number {
    const graphValues: number[] = map(this.data, (count: number) => Formatter.toHour(count));
    const maxValue: number | undefined = max(graphValues);
    const minValue: number = 2;
    if (typeof maxValue === 'undefined' || maxValue <= minValue) {
      return minValue;
    }
    const maxIntVal = Math.floor(maxValue);

    return (maxIntVal % 2 !== 0) ? maxIntVal + 1 : maxIntVal;
  }

  private static getVerticalScale(maxHour: number): { [key: string]: number }[] {
    const maxScaleTop: number = graphMaxHeight - graphItemMaxHeight;

    return [
      { hour: maxHour, top: maxScaleTop },
      { hour: maxHour / 2, top: maxScaleTop + graphItemMaxHeight / 2 },
      { hour: 0, top: graphMaxHeight - 1 },
    ];
  }

  private getGraphItemHeight(value: number): number {
    return graphItemMaxHeight / this.maxScale * Formatter.toHour(value);
  }

  private refleshContent(): void {
    this.ctx!.clearRect(0, 0, this.contentWidth, this.contentHeight);
  }
}
</script>

<style lang="scss" scoped>
.ReportsBarGraph {
  padding: 16px 0;

  &_Title {
    margin-left: 12px;
    font-size: 1.2rem;
    color: #8E8E93;
  }

  &_Content {
    width: 100%;
  }
}
</style>
