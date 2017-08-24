<template>
	<div class="img-wrap pr" v-loading.lock="imgLoading" :style="{height: dialogHeight - 62 + 'px'}">
		<img 
		class="img-target ps" 
		alt="图片加载中…"
		draggable="false"
		ondragstart="return false"
		:src="photoSrc + aliImgUrl"
		:style="{
			left: imgLeftValue + 'px',top: imgTopValue + 'px',
			width:sizeAccording ? 'auto' : imgTargetWidth + 'px',
			height: sizeAccording ? imgTargetHeight + 'px' : 'auto'}"
		@mousedown="mousedown($event)"
		@mousemove="figureLength($event)"
    @mouseup="clearDrag($event)"
    @mousewheel="getWheelScale($event)"
    @DOMMouseScroll="getWheelDalta($event)"
    @load="imgLoaded"
		>	
		<img
		class="img-ori" 
		:src="photoSrc + aliImgUrl"
		alt="">
		<div>
			<span class="ps zoom-big-btn iconfont icon-fangda" @click="zoomBig"></span>
		</div>
		
		<div>
			<span class="ps zoom-small-btn iconfont icon-minus" @click="zoomSmall"></span>
		</div>
		<span class="recover-img-btn ps" @click="recoverImgSize">原始尺寸</span>
		<em :class="showScale" class="scale-display ps">当前缩放比例 {{Math.round(scaleTimes*100)}}%</em>
	</div>
</template>
<script>
	export default {
		props:{
			photoSrc: {
				type: String,
				default: '../../../static/assets/img/home/banner3.jpg'
			},
			isDialogTransform: {
				type: Boolean,
				default: false
			},
			dialogHeight: {
				type: Number,
				default:500
			},
			dialogWidth: {
				type: Number,
				default:500
			},
			dialogTop: {
				type: Number,
				default: 500,
			},
			dialogLeft: {
				type: Number,
				default: 500,
			}
			
		},
		data(){
			return {
				maxScale: 2,
				minScale: 0.03,
				imgLeftValue: 800,
				imgTopValue: 800,
				scaleTimes: 1,
				scaleChange: 0.05,

				disX: 0,
				disY: 0,
				draging: false,

				scaleLockX:false,
				scaleLockY:false,

				isImgLoad: false,

				imgTargetWidth: 1000,
				imgTargetHeight: 1000,
				imgTarget: null,

				imgWrap:null,
				sizeAccording: false,

				imgLoading: false,
				imgLoadComplete: false,

				ZOOM_STEP: 0.03,

				fixedHeight: 0,
				fixedWidth:0,
				aliImgUrl: '',
				imgOriHeight: 0,
				imgOriWidth: 0,

				showScale: '',

				newImgLeft: 0,
				newImgTop: 0,

			}
		},
		mounted(){

    	this.imgTarget = document.getElementsByClassName("img-target")[0]
    	this.imgOri = document.getElementsByClassName("img-ori")[0]
			this.imgLoading = true
			this.imgWrap = document.getElementsByClassName("img-wrap")[0]
			this.judgeUserAgent()
		},
		methods:{
			getBigScaleImgPosition(oldLeft,oldTop,oldScale,oldImgWidth,oldImgHeight){

				var imgWrapWidth = this.imgWrap.clientWidth
				var imgWrapHeight = this.imgWrap.clientHeight
				var halfWrapHeight = this.imgWrap.clientHeight / 2
				var halfWrapWidth = imgWrapWidth / 2
				
				if (imgWrapHeight >= this.imgTarget.clientHeight || imgWrapWidth >= this.imgTarget.clientWidth || oldImgWidth < imgWrapWidth && imgWrapWidth <= this.imgTarget.clientWidth || oldImgHeight < imgWrapHeight && imgWrapHeight <= this.imgTarget.clientHeight) {
					this.setImgCenter()
				}else{
				this.imgLeftValue = halfWrapWidth - this.scaleTimes * (halfWrapWidth - oldLeft) / oldScale 

				this.imgTopValue = halfWrapHeight - this.scaleTimes * (halfWrapHeight - oldTop) / oldScale 
				}
				this.imgFillWrap(imgWrapWidth,imgWrapHeight)
			},
			getSmallScaleImgPosition(oldLeft,oldTop,oldScale,oldImgWidth,oldImgHeight){

				var imgWrapWidth = this.imgWrap.clientWidth
				var imgWrapHeight = this.imgWrap.clientHeight
				var halfWrapHeight = this.imgWrap.clientHeight / 2
				var halfWrapWidth = imgWrapWidth / 2
			
				if(oldImgWidth > imgWrapWidth && imgWrapWidth >= this.imgTarget.clientWidth && oldImgHeight > imgWrapHeight && imgWrapHeight >= this.imgTarget.clientHeight){
					this.setImgCenter()
				}else{
				this.imgLeftValue = halfWrapWidth - this.scaleTimes * (halfWrapWidth - oldLeft) / oldScale 

				this.imgTopValue = halfWrapHeight - this.scaleTimes * (halfWrapHeight - oldTop) / oldScale 
				}
				this.imgFillWrap(imgWrapWidth,imgWrapHeight)
			},
			imgFillWrap(imgWrapWidth,imgWrapHeight){
				//不出现白边
				if (imgWrapWidth < this.imgTarget.clientWidth && this.imgLeftValue > 0 ) {
					this.imgLeftValue = 0
				}else{
					this.imgTopValue = this.imgTopValue
				}
				if (imgWrapHeight < this.imgTarget.clientHeight && this.imgTopValue > 0) {
					this.imgTopValue = 0
				}else{
					this.imgTopValue = this.imgTopValue
				}
				if (imgWrapWidth < this.imgTarget.clientWidth && this.imgTarget.clientWidth - imgWrapWidth < -this.imgLeftValue) {
					this.imgLeftValue = -(this.imgTarget.clientWidth - imgWrapWidth)
				}else{
					this.imgTopValue = this.imgTopValue
				}
				if (imgWrapHeight < this.imgTarget.clientHeight && this.imgTarget.clientHeight - imgWrapHeight < -this.imgTopValue) {
					this.imgTopValue = -(this.imgTarget.clientHeight - imgWrapHeight)
				}else{
					this.imgTopValue = this.imgTopValue
				}
				
				if (imgWrapHeight > this.imgTarget.clientHeight) {
					this.setImgCenter()
				}else if (imgWrapWidth > this.imgTarget.clientWidth) {
					this.setImgCenter()
				}else{
					return
				}
			},
			getScaleScrollImgPosition(ev,oldLeft,oldTop,oldScale){

				var centerX = ev.clientX - this.dialogLeft - 52
				var centerY = ev.clientY - this.dialogTop - 52
				var imgWrapWidth = this.imgWrap.clientWidth
				var imgWrapHeight = this.imgWrap.clientHeight
			
				if (this.imgTarget.clientWidth <= imgWrapWidth && this.imgTarget.clientHeight <= imgWrapHeight) {
					this.minScale = this.scaleTimes
					this.setImgCenter()
					return
				}
				else{
					this.imgLeftValue = centerX - this.scaleTimes * (centerX - oldLeft) / oldScale 
					this.imgTopValue = centerY - this.scaleTimes * (centerY - oldTop) / oldScale 
				}
				this.imgFillWrap(imgWrapWidth,imgWrapHeight)
			},
			displayScaleStatus(){
				clearTimeout(this.showPopTimer) 
				this.showScale = 'transimg-pop-animation-in'
				this.showPopTimer = setTimeout(() => {
					this.showScale = 'transimg-pop-animation-out'
				},1500)
			},
			judgeUserAgent(){
				var ua = navigator.userAgent.toLowerCase()
				if (ua.includes('firefox')) {
					this.aliImgUrl = '?x-oss-process=image/resize,w_2000'
				}else{
					this.aliImgUrl = '?x-oss-process=image/auto-orient,1'
				}
			},
			getWheelDalta(ev){
				ev.preventDefault()
				var delta = -ev.detail / 3 * 120;
				if (delta > 0 && this.scaleTimes <= this.maxScale || delta < 0 && this.scaleTimes >= this.minScale) {
      		var changed = delta * 0.0002
      		if (this.scaleTimes + changed >= this.maxScale && changed > 0) {
      			var oldLeft = this.imgLeftValue,
								oldTop = this.imgTopValue,
								oldScale = this.scaleTimes
      			this.setMaxSize()
      			this.getScaleScrollImgPosition(ev,oldLeft,oldTop,oldScale)
      		} else if (this.scaleTimes + changed <= this.minScale && changed < 0) {
      			this.setMinSize()
      			this.setImgCenter()
      		}else {
      			var oldLeft = this.imgLeftValue,
								oldTop = this.imgTopValue,
								oldScale = this.scaleTimes

      			this.scaleTimes += changed
      			this.scaleTimes = this.scaleTimes < this.minScale ? this.minScale : this.scaleTimes
      			this.scaleTimes = this.scaleTimes > this.maxScale ? this.maxScale : this.scaleTimes

      			if (this.sizeAccording) {
							this.imgTargetHeight = this.fixedHeight * this.scaleTimes

						} else{
							this.imgTargetWidth = this.fixedWidth * this.scaleTimes
						}
      			
      			this.getScaleScrollImgPosition(ev,oldLeft,oldTop,oldScale)
      			console.log(changed)
      		}
      	}
      	this.displayScaleStatus()
			},
			zoomByWidth(){
				this.imgTargetWidth = this.fixedWidth * (this.scaleTimes - this.scaleChange)
				this.scaleTimes -= this.ZOOM_STEP
			},
			zoomByHeight(){
				this.imgTargetHeight = this.fixedHeight * (this.scaleTimes - this.scaleChange)
				this.scaleTimes -= this.ZOOM_STEP
			},
			setImgCenter(){
				this.$nextTick(() => {
					this.imgTopValue = (this.imgWrap.clientHeight - this.imgTarget.clientHeight - 0) / 2
					this.imgLeftValue = (this.imgWrap.clientWidth - this.imgTarget.clientWidth - 0) / 2
				})
			},
			zoomBig(){
				var oldLeft = this.imgLeftValue,
						oldTop = this.imgTopValue,
						oldScale = this.scaleTimes,
						oldImgWidth = this.imgTarget.clientWidth,
						oldImgHeight = this.imgTarget.clientHeight

				if (this.scaleTimes + this.scaleChange >= this.maxScale) {
					this.scaleTimes = this.maxScale
					if (this.sizeAccording) {
						this.imgTargetHeight = this.fixedHeight * this.scaleTimes

					} else{
						this.imgTargetWidth = this.fixedWidth * this.scaleTimes
					}
					this.getBigScaleImgPosition(oldLeft,oldTop,oldScale,oldImgWidth,oldImgHeight)
				}else{
					this.accordingSizeZoomBig()
					this.scaleTimes += this.scaleChange
					this.$nextTick(() => {
						this.getBigScaleImgPosition(oldLeft,oldTop,oldScale,oldImgWidth,oldImgHeight)
					})
				}
				this.displayScaleStatus()
			},
			accordingSizeZoomBig(){
				if (this.sizeAccording) {
					this.imgTargetHeight = this.fixedHeight * (this.scaleTimes + this.scaleChange)
				}else{
					this.imgTargetWidth = this.fixedWidth * (this.scaleTimes + this.scaleChange)
				}
			},
			zoomSmall(ev){
				var oldLeft = this.imgLeftValue,
						oldTop = this.imgTopValue,
						oldScale = this.scaleTimes,
						oldImgWidth = this.imgTarget.clientWidth,
						oldImgHeight = this.imgTarget.clientHeight

				if (this.scaleTimes - this.scaleChange <= this.minScale) {
					this.scaleTimes = this.minScale
					if (this.sizeAccording) {
						this.imgTargetHeight = this.fixedHeight * this.scaleTimes

					} else{
						this.imgTargetWidth = this.fixedWidth * this.scaleTimes
					}
					this.setImgCenter()
				}else{
					this.accordingSizeZoomSmall()
					this.scaleTimes -= this.scaleChange
					this.$nextTick(() => {
						this.getSmallScaleImgPosition(oldLeft,oldTop,oldScale,oldImgWidth,oldImgHeight)
					})
				}
				this.displayScaleStatus()
				
			},
			recoverImgSize(){
				var oldLeft = this.imgLeftValue,
						oldTop = this.imgTopValue,
						oldScale = this.scaleTimes,
						oldImgWidth = this.imgTarget.clientWidth,
						oldImgHeight = this.imgTarget.clientHeight

				if (this.sizeAccording) {
					this.scaleTimes = 1
					this.imgTargetHeight = this.imgOriHeight
				} else{
					this.scaleTimes = 1
					this.imgTargetWidth = this.imgOriWidth
				}
				this.displayScaleStatus()
				this.$nextTick(() => {
					this.getBigScaleImgPosition(oldLeft,oldTop,oldScale,oldImgWidth,oldImgHeight)
				})
			},
			accordingSizeZoomSmall(){
					if (this.sizeAccording) {
						this.imgTargetHeight = this.fixedHeight * (this.scaleTimes - this.scaleChange)
					}else{
						this.imgTargetWidth = this.fixedWidth * (this.scaleTimes - this.scaleChange)
					}
			},
			mousedown(event){
        event = event || window.event; 
        this.disX = event.clientX
        this.disY = event.clientY
        this.draging = true
      },
      figureLength(event) {
       	var event = event || window.event
        if (this.draging) {
      		this.getScaleLockStatus()
	        if (this.scaleLockX && this.scaleLockY === false) {
	          this.moveHorizontal(event)
	        }else if (this.scaleLockY && this.scaleLockX === false) {
	          this.moveVertical(event)
	        }else if (this.scaleLockX && this.scaleLockY) {
	        	this.moveFree(event)
	        }else{
	        	return
	        }
      	}
        
      },
      moveHorizontal(event){
      	var moveLeft = event.clientX - this.disX;  

	      this.imgLeftValue += moveLeft
        this.disX = event.clientX
				this.judgeDragBoundaryHorizontal()
      },
      moveVertical(event){
      	var moveTop = event.clientY - this.disY;

        this.imgTopValue += moveTop
        this.disY = event.clientY
				this.judgeDragBoundaryVertical()
      },
      moveFree(event){
      	var moveLeft = event.clientX - this.disX
        var moveTop = event.clientY - this.disY
  
        this.imgLeftValue += moveLeft
        this.imgTopValue += moveTop 
        this.disX = event.clientX
        this.disY = event.clientY
				this.judgeDragBoundaryVertical()
				this.judgeDragBoundaryHorizontal()
      },
      judgeDragBoundaryHorizontal(){
      		if (this.imgLeftValue > 0) {
	      		this.imgLeftValue = 0
	      	}else if (this.imgLeftValue < this.imgWrap.clientWidth - this.imgTarget.clientWidth) {
		      	this.imgLeftValue = this.imgWrap.clientWidth - this.imgTarget.clientWidth
		      }else{
		      	this.imgLeftValue = this.imgLeftValue
		      }

		      if (this.imgWrap.clientWidth > this.imgTarget.clientWidth) {
		      	this.setImgCenter()
		      }else{
		      	return
		      }
      },
      judgeDragBoundaryVertical(){
      	if (this.imgTopValue > 0) {
      		this.imgTopValue = 0
	     	}else if (this.imgTopValue < this.imgWrap.clientHeight - this.imgTarget.clientHeight - 10) {
	     		this.imgTopValue = this.imgWrap.clientHeight - this.imgTarget.clientHeight - 10
	     	}else{
	     		this.imgTopValue = this.imgTopValue
	     	}
	     	if (this.imgWrap.clientHeight > this.imgTarget.clientHeight) {
		      	this.setImgCenter()
		      }else{
		      	return
		      }
      },
      clearDrag() {
        this.draging = false
      },
      getScaleLockStatus(){
      	var imgWrapHeight = this.imgWrap.clientHeight
      	var imgWrapWidth = this.imgWrap.clientWidth
      	
      	setTimeout(() => {
      		var imgWidth = this.imgTarget.clientWidth
      		var imgHeight = this.imgTarget.clientHeight

      		if (imgWidth <= imgWrapWidth && imgHeight >= imgWrapHeight) {
	      		this.scaleLockX = false
	      		this.scaleLockY = true
	      	}else if (imgWidth >= imgWrapWidth && imgHeight <= imgWrapHeight){
	      		this.scaleLockX = true
	      		this.scaleLockY = false
	      	}else if (imgWidth >= imgWrapWidth && imgHeight >= imgWrapHeight){
	      		this.scaleLockX = true
	      		this.scaleLockY = true
	      	}else{
	      		this.scaleLockX = false
	      		this.scaleLockY = false
	      	}

      	})
      },
      getWheelScale(ev){
      	ev.preventDefault()
      	if (ev.wheelDelta > 0 && this.scaleTimes <= this.maxScale || ev.wheelDelta < 0 && this.scaleTimes >= this.minScale) {
      		var changed = ev.wheelDelta * 0.0002
      		if (this.scaleTimes + changed >= this.maxScale && changed > 0) {
      			var oldLeft = this.imgLeftValue,
								oldTop = this.imgTopValue,
								oldScale = this.scaleTimes
      			this.setMaxSize()
      			this.getScaleScrollImgPosition(ev,oldLeft,oldTop,oldScale)
      		} else if (this.scaleTimes + changed <= this.minScale && changed < 0) {
      			this.setMinSize()
      			this.setImgCenter()
      		} else {
      			var oldLeft = this.imgLeftValue,
								oldTop = this.imgTopValue,
								oldScale = this.scaleTimes,
								oldImgWidth = this.imgTarget.clientWidth,
								oldImgHeight = this.imgTarget.clientHeight

      			this.scaleTimes += changed
      			this.scaleTimes = this.scaleTimes < this.minScale ? this.minScale : this.scaleTimes
      			this.scaleTimes = this.scaleTimes > this.maxScale ? this.maxScale : this.scaleTimes

      			if (this.sizeAccording) {
							this.imgTargetHeight = this.fixedHeight * this.scaleTimes

						} else{
							this.imgTargetWidth = this.fixedWidth * this.scaleTimes
						}

      			this.getScaleScrollImgPosition(ev,oldLeft,oldTop,oldScale)

      		}
      	}
      	this.displayScaleStatus()
      },
      setMinSize(){
      	this.scaleTimes = this.minScale
				if (this.sizeAccording) {
					this.imgTargetHeight = this.fixedHeight * this.scaleTimes

				} else{
					this.imgTargetWidth = this.fixedWidth * this.scaleTimes
				}
      },
      setMaxSize(){
      	this.scaleTimes = this.maxScale
				if (this.sizeAccording) {
					this.imgTargetHeight = this.fixedHeight * this.scaleTimes

				} else{
					this.imgTargetWidth = this.fixedWidth * this.scaleTimes
				}
      },
    	imgLoaded(){
    		this.imgOriHeight = this.imgOri.clientHeight
    		this.imgOriWidth = this.imgOri.clientWidth
    		this.imgTopValue = (this.imgWrap.clientHeight - this.imgTarget.clientHeight) / 2
				this.imgLeftValue = (this.imgWrap.clientWidth - this.imgTarget.clientWidth) / 2
    		this.$nextTick(() => {
    			
    			this.setFixedSize()
	    		this.sizeAccording = false
					var count = 0
					let CLIENT_WIDTH = this.imgWrap.clientWidth
					let CLIENT_HEIGHT = this.imgWrap.clientHeight

					var wrapRate = CLIENT_WIDTH / CLIENT_HEIGHT

					var imgRate = this.wrapWidth / this.imgTarget.clientHeight

					if (wrapRate < imgRate) {
						while ( CLIENT_WIDTH <= this.imgTargetWidth && count < 1000){
							this.zoomByWidth()
							count ++
						}
					} else {
						this.sizeAccording = true

						while ( CLIENT_HEIGHT <= this.imgTargetHeight && count < 1000){
							this.zoomByHeight()
							count ++
						}
					}
					this.minScale = this.scaleTimes
					this.setImgCenter()
					this.imgLoading = false
    		})
    	},
    	initImgSize() {
				this.imgTargetHeight = this.imgWrap.clientHeight
    	},
	    setFixedSize() {
	    		this.fixedHeight = this.imgOriHeight
					this.fixedWidth = this.imgOriWidth
	    },
		},
		watch:{
			photoSrc(url){
				if (url) {
					this.judgeUserAgent()
					this.isImgLoad = true
				}
			},
			isDialogTransform(transform){
				if (transform) {
					this.setImgCenter()
					this.$emit('initTransform',false)
				}
			},
			dialogHeight(height){
				if (height) {
					if (this.imgTargetHeight > height) {
						return
					}else if (this.imgTargetHeight < height - 140) {
						return
					}else{
						this.imgTargetHeight = height - 90
						this.scaleTimes = this.imgTargetHeight / this.fixedHeight
					}
				}
			},
			dialogWidth(width){
				if (width) {
					if (this.imgTarget.clientWidth > width) {
						return
					}else if (this.imgTarget.clientWidth < width) {
						return
					}else{
						this.imgTarget.imgTargetWidth = width
						this.scaleTimes = this.imgTargetWidth / this.fixedWidth
					}
				}
			},
			imgTargetHeight(change){
				if (change) {
					if (change < this.fixedHeight * this.minScale) {
						this.imgTargetHeight = this.fixedHeight * this.minScale
						this.setImgCenter()
					}
				}
			},
			imgTargetWidth(change){
				if (change) {
					if (change < this.fixedWidth * this.minScale) {
						this.imgTargetWidth = this.fixedWidth * this.minScale
						this.setImgCenter()
					}
				}
			},
		}
	}
</script>
<style
 lang="less" scoped>
	@import '../../assets/style/common';
	.img-wrap {
		padding: 10px;
		width: 100%;
		box-sizing: border-box;
		border-top: 1px solid #cbcbcb;
		background: @assistant-bg;
  	user-select: none;
  	overflow: hidden;
  		.img-target {
	  		top: 0;
	  		left: 0;
	  		cursor: move;
	  		width: 100%;
	  	}
	  	.img-ori{
	  		top: 0;
	  		left: 0;
	  		position: absolute;
	  		top: 0;
	  		left: 0;
	  		z-index: -1000;
	  		opacity: 0;
	  	}
  	
  	
		.zoom-big-btn {
			.adv-height(30px);
			.adv-font-large();
			.adv-border-right-radius();
			display: inline-block;
			width: 30px;
			text-align: center;
			top: 12px;
			right: 10px;
			cursor: pointer;
			background: #fff;
			border: 1px solid @border-color;
			
			&:hover{
				color: @theme-hover-fc;
			};
		}
		.zoom-small-btn {
			.adv-height(30px);
			.adv-font-large();
			.adv-border-left-radius();
			display: inline-block;
			width: 30px;
			text-align: center;
			top: 12px;
			right: 40px;
			cursor: pointer;
			background: #fff;
			border: 1px solid @border-color;
			&:hover{
				color: @theme-hover-fc;
			};
		}
		.recover-img-btn {
			height: 32px;
			line-height: 30px;
			.adv-font-normal();
			box-sizing: border-box;
			padding: 0 14px;
			border-radius: 4px;
			display: inline-block;
			text-align: center;
			top: 12px;
			right: 80px;
			color: #8f8d8b;
			cursor: pointer;
			background: #fff;
			border: 1px solid @border-color;
			&:hover{
				color: @theme-hover-fc;
			};
		}
		.scale-display {
			.adv-height(30px);
			.adv-font-small();
			display: inline-block;
			background: black;
			border-radius: 4px;
			padding:0 16px;
			opacity: 0;
			color: white;
			top: 93%;
  		left: 50%;
  		.adv-trans-translate(-50%,-50%);
		}

		.transimg-pop-animation-out {
	   .adv-animation(popfadeout;.38s;1;forwards);
	  }
	  .transimg-pop-animation-in {
	   .adv-animation(popfadein;0.001s;1;forwards);
	  }
		
		@keyframes popfadeout {
		  0% {
		   	opacity: 0.6;
		  }
		  
		  100% {
		    opacity: 0;
		  }
		}

		@keyframes popfadein {
		  0% {
		   	opacity: 0;
		  }
		  100% {
		    opacity: 0.6;
		  }
		}
	}

</style>