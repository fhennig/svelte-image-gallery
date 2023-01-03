<script lang="ts">
  import type { TItem } from '$lib/types';
  import LeftNav from '$lib/LeftNav.svelte';
  import RightNav from '$lib/RightNav.svelte';
  import { createEventDispatcher } from 'svelte';
  import Slide from '$lib/Slide.svelte';

  export let slideWrapperClass: string;
  export let items: TItem[];
  export let showNav: boolean;
  export let showBullets: boolean;
  export let showIndex: boolean;
  export let currentIndex: number;
  export let infinite: boolean;
  export let isRTL: boolean;

  $: canSlide = items.length >= 2;
  $: canSlidePrevious = currentIndex > 0;
  $: canSlideNext = currentIndex < items.length - 1;
  $: canSlideLeft = infinite || (isRTL ? canSlideNext : canSlidePrevious);
  $: canSlideRight = infinite || (isRTL ? canSlidePrevious : canSlideNext);

  let currentSlideOffset = 0;

  const dispatch = createEventDispatcher();

  function getAlignmentClassName(index) {
    // Necessary for lazing loading
    let alignment = '';
    const leftClassName = 'image-gallery-left';
    const centerClassName = 'image-gallery-center';
    const rightClassName = 'image-gallery-right';

    switch (index) {
      case currentIndex - 1:
        alignment = ` ${leftClassName}`;
        break;
      case currentIndex:
        alignment = ` ${centerClassName}`;
        break;
      case currentIndex + 1:
        alignment = ` ${rightClassName}`;
        break;
      default:
        break;
    }

    if (items.length >= 3 && infinite) {
      if (index === 0 && currentIndex === items.length - 1) {
        // set first slide as right slide if were sliding right from last slide
        alignment = ` ${rightClassName}`;
      } else if (index === items.length - 1 && currentIndex === 0) {
        // set last slide as left slide if were sliding left from first slide
        alignment = ` ${leftClassName}`;
      }
    }

    return alignment;
  }

  function getSlideStyle(index) {
    const { currentIndex, currentSlideOffset, slideStyle } = this.state;
    const { infinite, items, useTranslate3D, isRTL } = this.props;
    const baseTranslateX = -100 * currentIndex;
    const totalSlides = items.length - 1;

    // calculates where the other slides belong based on currentIndex
    // if it is RTL the base line should be reversed
    let translateX = (baseTranslateX + index * 100) * (isRTL ? -1 : 1) + currentSlideOffset;

    if (infinite && items.length > 2) {
      if (currentIndex === 0 && index === totalSlides) {
        // make the last slide the slide before the first
        // if it is RTL the base line should be reversed
        translateX = -100 * (isRTL ? -1 : 1) + currentSlideOffset;
      } else if (currentIndex === totalSlides && index === 0) {
        // make the first slide the slide after the last
        // if it is RTL the base line should be reversed
        translateX = 100 * (isRTL ? -1 : 1) + currentSlideOffset;
      }
    }

    // Special case when there are only 2 items with infinite on
    if (infinite && items.length === 2) {
      translateX = this.getTranslateXForTwoSlide(index);
    }

    let translate = `translate(${translateX}%, 0)`;

    if (useTranslate3D) {
      translate = `translate3d(${translateX}%, 0, 0)`;
    }

    // don't show some slides while transitioning to avoid background transitions
    const isVisible = this.isSlideVisible(index);

    return {
      display: isVisible ? 'inherit' : 'none',
      WebkitTransform: translate,
      MozTransform: translate,
      msTransform: translate,
      OTransform: translate,
      transform: translate,
      ...slideStyle
    };
  }
</script>

<div class={slideWrapperClass}>
  <!-- TODO: render custom controls -->
  {#if canSlide}
    {#if showNav}
      <LeftNav on:click={dispatch('slideleft')} disabled={!canSlideLeft} />
      <RightNav on:click={dispatch('slideright')} disabled={!canSlideRight} />
    {/if}
    <!--TODO: SwipeWrapper-->
    <div class="image-gallery-slides">
      {#each items as item, index}
        <Slide
          {index}
          alignment={getAlignmentClassName(index)}
          originalClass={item.originalClass}
          slideStyle=""
          showItem={true}
          {item}
          isFullscreen={false}
        />
      {/each}
    </div>
  {:else}
    <div class="image-gallery-slides">
      {#each items as item, index}
        <Slide
          {index}
          alignment={getAlignmentClassName(index)}
          originalClass={item.originalClass}
          slideStyle=""
          showItem={true}
          {item}
          isFullscreen={false}
        />
      {/each}
    </div>
  {/if}

  <!-- TODO: render play button -->

  <!--{#if showBullets}-->
  <!--  <div class="image-gallery-bullets">-->
  <!--    <div-->
  <!--      class="image-gallery-bullets-container"-->
  <!--      role="navigation"-->
  <!--      aria-label="Bullet Navigation"-->
  <!--    >-->
  <!--      {bullets}-->
  <!--    </div>-->
  <!--  </div>-->
  <!--{/if}-->
  <!-- TODO render fullscreen button -->
  {#if showIndex}
    <div class="image-gallery-index">
      <span class="image-gallery-index-current">
        {currentIndex + 1}
      </span>
      <span class="image-gallery-index-separator">
        {' / '}
      </span>
      <span class="image-gallery-index-total">
        {items.length}
      </span>
    </div>
  {/if}
</div>