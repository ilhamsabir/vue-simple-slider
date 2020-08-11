<template>
	<div
		:style="style"
		:class="[{'image-slider--single': images.length === 1 }, `transition-${transition}`]"
		class="image-slider">
		<!-- Items -->
		<div
			class="image-slider-items">
			<a
				v-for="(item, i) in images"
				:key="i"
				:class="getClassName(i)"
				:draggable="true"
				:href="item.link"
				class="slide"
				@touchstart="onDragStart"
				@touchend="onDragStop">
				<picture :key="i">
					<img :src="item.image" alt="" @load="onImageLoad(i)">
				</picture>
			</a>
		</div>

		<!-- Nav -->
		<div
			v-if="showNav"
			class="image-slider__nav">
			<div
				class="prev"
				href="#"
				@click="prev">&#10094;</div>
			<div
				class="next"
				href="#"
				@click="next">&#10095;</div>
		</div>

		<!-- Dots -->
		<div
			v-if="dots"
			class="image-slider__dots">
			<div
				v-for="(image, index) in images"
				:key="index"
				:class="{ 'active': index === currentIndex }"
				class="image-slider__dot"
				@click="goTo(index)"/>
		</div>
	</div>
</template>

<script>
export default {
	props: {
		images: {
			type: Array,
			default: () => [],
		},
		interval: {
			type: Number,
			default: 5000,
		},
		autoplay: {
			type: Boolean,
			default: true,
		},
		dots: {
			type: Boolean,
			default: true,
		},
		showNav: {
			type: Boolean,
			default: true,
		},
		transition: {
			type: String,
			default: 'fade',
		},
	},
	data() {
		return {
			slideIndex: 1,
			timer: null,
			currentIndex: 0,
			style: {
				minHeight: '0',
			},
			heights: [],
			offsetX: 0,
			dragTreshold: 100,
			nextIndex: 1,
			prevIndex: 1,
			imgPlaceholder: '',
		}
	},
	computed: {
		isMultipleBanner() {
			return this.images.length > 1
		}
	},
	watch: {
		currentIndex(index) {
			if (this.heights[index]) {
				const height = this.heights[index]
				this.style.minHeight = `${height}px`
			}
		},
	},
	created() {
		if (this.isMultipleBanner) {
			this.prevIndex = this.images.length - 1
		}
	},
	mounted() {
		if (this.autoplay && this.isMultipleBanner) {
			this.startSlide()
		}

		window.addEventListener('resize', this.onImageLoad(this.currentIndex));
	},
	destroyed() {
		window.removeEventListener('resize', this.onImageLoad(this.currentIndex));
	},
	methods: {

		getClassName(i) {
			const className = {
				'active': i === this.currentIndex,
				'inactive': i !== this.currentIndex,
				'prev-item': i === this.prevIndex,
				'next-item': i === this.nextIndex,
			}
			return className
		},

		startSlide() {
			if (this.autoplay) {
				const interval = this.interval * 1000
				this.timer = setInterval(this.next, interval)
			}
		},

		resetTimer() {
			clearInterval(this.timer)
			this.startSlide()
		},

		next() {
			this.resetTimer()
			const nextIndex = this.currentIndex >= (this.images.length - 1) ? 0 : this.currentIndex + 1
			this.currentIndex = nextIndex
			this.setPrevNextIndex()
		},

		prev() {
			this.resetTimer()
			const prevIndex = this.currentIndex === 0 ? (this.images.length - 1) : this.currentIndex - 1
			this.currentIndex = prevIndex
			this.setPrevNextIndex()
		},

		setPrevNextIndex() {
			const totalImages = this.images.length
			const index = this.currentIndex
			let next = index
			if ((index + 1) > (totalImages - 1)) {
				next = 0
			} else if ((index + 1) === (totalImages - 1)) {
				next = index + 1
			} else {
				next = index + 1
			}

			let prev = 0
			if (index - 1 < 0) {
				prev = totalImages - 1
			} else if (index - 1 === 0) {
				prev = 0
			} else {
				prev = index - 1
			}

			this.nextIndex = next
			this.prevIndex = prev
		},

		onImageLoad(index) {
			let height = this.$el.offsetHeight
			height = this.$el.querySelector('.active') ? this.$el.querySelector('.active').offsetHeight : height
			this.$set(this.heights, index, height)
			this.style.minHeight = `${height}px`
		},

		goTo(index) {
			this.resetTimer()
			this.currentIndex = index
			this.setPrevNextIndex()
		},

		onDragStart(e) {
			if (this.isMultipleBanner) {
				const touches = e.changedTouches
				const firstTouch = touches[0]
				const currentX = firstTouch.clientX
				this.offsetX = currentX
			}
		},

		onDragStop(e) {
			if (this.isMultipleBanner) {
				const dragTreshold = this.dragTreshold
				const touches = e.changedTouches
				const firstTouch = touches[0]
				const currentX = firstTouch.clientX
				if (this.offsetX !== 0) {
					const difference = this.offsetX - currentX
					// slide left
					if (difference > 0 && difference > dragTreshold) {
						this.next()
					}
					// slide right
					else if (difference < 0 && (difference * -1) > dragTreshold) {
						this.prev()
					}
				}
				this.offsetX = currentX
			}
		},
	},
}
</script>

<style lang="scss">
.image-slider {
	position: relative;
	min-height: 50px;
	overflow: hidden;

	&--single {
		.image-slider__dots,
		.image-slider__nav {
			display: none;
		}
	}

	.prev,
	.next {
		position: absolute;
		top: 50%;
		cursor: pointer;
		width: auto;
		padding: 16px;
		color: white;
		font-weight: bold;
		font-size: 18px;
		transition: 0.7s ease;
		text-decoration: none;
		user-select: none;
		transform: translateY(-50%);
		background-color: rgba(102, 102, 102, 0.2);
		z-index: 3;
	}

	/* Position the "next button" to the right */
	.next {
		right: 0;
	}
	.prev {
		left: 0;
	}
	/* On hover, add a black background color with a little bit see-through */
	.prev:hover,
	.next:hover {
		background-color: rgba(0,0,0,0.9);
	}
}

.image-slider-items {
	position: relative;
	overflow: hidden;
	// transform-style: preserve-3d;
}

// .slide-fade {
// 	&.enter {
// 		opacity: 1;
// 	}

// 	&.leave {
// 		opacity: 0;
// 		height: 0;
// 	}
// }

.slide {
	// position: absolute;
	// top: 0;
	display: block;
	margin: auto;
	width: 100%;
	transition: .3s all ease-in-out;
	z-index: 1;
	opacity: 0;

	&.active {
		opacity: 1;
		position: relative;
		z-index: 2;
	}

	img {
		width:100%;
		min-height: 99px;
	}
}

.transition-fade {
	.inactive {
		height: 0;
		opacity: 0;
	}
}

.transition-slide {
	.slide {
		position: absolute;
		top: 0;
		transition: transform .5s, opacity .5s, z-index .5s;

		&.active {
			position: relative;
		}
	}

	.next-item {
		transform: translateX(100%);
	}

	.prev-item {
		transform: translateX(-100%);
	}
}

.slide-leave-active,
.slide-enter-active {
	transition: 1s;
}

.slide-enter {
	/* transform: translateX(0); */
	// animation: slideIn 1s 1;
	opacity: 1;
}

.slide-leave {
	opacity: 0;
	height: 0;
	/* transform: translateX(-100%); */
	// animation: slideOut 1s 1;
}

// .slide.hidden {
// 	display: none;
// }

.slide .loader {
	position: absolute;
	top: 0;
	left: 0;
	bottom: 0;
	right: 0;
	margin: auto;
	min-height: 4rem;
}

.img-slider {
	overflow: hidden;
	position: relative;
	height: 200px;
	width: 100%;
}

.image-slider__dots {
	position: absolute;
	bottom: 0;
	left: 0;
	right: 0;
	margin: auto;
	display: flex;
	justify-content: center;
	align-items: center;
	padding-bottom: 1rem;
}

.image-slider__dot {
	background: rgba(193, 193, 193, 0.7);
	border-radius: 50%;
	border: 1px solid #fff;
	width: 12px;
	height: 12px;
	margin-right: 5px;
	cursor: pointer;
	z-index: 3;

	&:hover,
	&.active {
		background: #666;
	}
}
</style>
