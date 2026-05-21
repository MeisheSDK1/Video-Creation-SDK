<template>
    <div class="main" v-loading="isSdkLoading">
        <video id="preview" v-show="!isImageMedia"></video>
        <canvas id="image-preview" v-show="isImageMedia"></canvas>
        <div class="inputs">
            <button @click="useCameraMediaStream">CameraMediaStream</button>
            <button @click="useVideoMediaStream">VideoMediaStream</button>
            <button @click="useImageMediaStream">ImageMediaStream</button>
            <button @click="useImageData">ImageData</button>
        </div>
        <div class="features">
            <button @click="enableBeautyClick">{{ enableBeautyBtnText }}</button>
            <button @click="addShape">Shape</button>
            <button @click="addMakeup">Makeup</button>
            <button @click="addSegmentationImageBackground">ImageBackground</button>
            <button @click="addSegmentationBlurBackground">BlurBackground</button>
            <button @click="addAvatar">Avatar</button>
            <button @click="addCompoundMakeups">CompoundMakeups</button>
            <button @click="addVideoFx">VideoFx</button>
        </div>
        <div class="features">
            <button @click="addCaption">Caption</button>
            <button @click="addModularCaption">ModularCaption</button>
            <button @click="addCompoundCaption">CompoundCaption</button>
            <button @click="addAnimatedSticker">AnimatedSticker</button>
        </div>
    </div>
</template>
<script>
import { WASMLoader } from 'meishewasmloader'
import jszip from 'jszip'

export default {
    name: 'EffectSdkDemo',
    data() {
        return {
            mediaStream: null,
            isImageMedia: false,
            mirror: false,
            arSceneRenderer: null,
            preview: null,
            isSdkLoading: false,
            enableBeauty: true,
            externalEffectInstanceArray: [],
        }
    },
    computed: {
        enableBeautyBtnText() {
            return this.enableBeauty ? 'Turn Off Beauty' : 'Turn On Beauty';
        }
    },
    watch: {
        isImageMedia(newValue, oldValue) {
            this.initPreview(newValue);
        },
    },
    async mounted() {
        this.isSdkLoading = true;
        this.initPreview(false);
        try {
            await this.loadEffectSDK();
            await this.useCameraMediaStream();
        } catch (e) {
            console.error(e);
            this.preview.srcObject = this.mediaStream;
            this.preview.play();
        }
        this.isSdkLoading = false;
    },
    unmounted() {
        this.releaseResources();
    },
    methods: {
        initPreview(imagePreview) {
            const width = 960;
            const height = 540;
            if (imagePreview) {
                this.preview = document.getElementById('image-preview');
            } else {
                this.preview = document.getElementById('preview');
            }
            this.preview.style.width = width + "px";
            this.preview.style.height = height + "px";
            this.preview.width = width * window.devicePixelRatio;
            this.preview.height = height * window.devicePixelRatio;
            window.addEventListener('resize', function() {
                if (this.preview) {
                    this.preview.style.width = width + "px";
                    this.preview.style.height = height + "px";
                    this.preview.width = width * window.devicePixelRatio;
                    this.preview.height = height * window.devicePixelRatio;
                }
            }, true);
        },
        loadEffectSDK() {
            return new Promise(async (resolve, reject) => {
                if (Module.Meishe && Module.Meishe.effectContext) {
                    resolve();
                    return;
                }
                this.wasmLoader = WASMLoader({
                    showError(errorText) {
                        reject(new Error('Failed to load wasm' + errorText));
                    },
                    loadingFinished: async () => {
                        function poll() {
                            if (Module.Meishe !== undefined && Module.Meishe.effectContext !== undefined) {
                            // if (nveGetEffectContextInstance().verifySdkLicenseFileUrl('')) {
                            resolve();
                            // } else {
                            //     reject(new Error('Failed to verify sdk license!'));
                            // }
                            } else {
                            setTimeout(() => poll(resolve, reject), 400)
                            }
                        }
                        poll()
                    }
                });
                this.wasmLoader.loadEmscriptenModule('https://alieasset.meishesdk.com/NvWasm/domain/3-15-1-release/2/', { effectSdk: true });
            });
        },
        async initARSceneRenderer() {
            this.enableBeauty = true;
            this.arSceneRenderer = new NveARSceneRenderer();
            await this.arSceneRenderer.init({
                faceModelUrl: "https://alieasset.meishesdk.com/model/face240/ms_face240_v3_0_1_next.model",
                eyecontourModelUrl: "https://alieasset.meishesdk.com/model/eyecontour/ms_eyecontour_v2_0_0_next.model",
                avatarModelUrl: "https://alieasset.meishesdk.com/model/avatar/ms_avatar_v2_0_0_next.model",
                segmentationModelUrl: "https://alieasset.meishesdk.com/model/humanseg/ms_humansegment_medium_v2_0_0_next.model",
                makeupDataUrl: "https://alieasset.meishesdk.com/model/makeup2_240_v2.1.2.dat",
                fakefaceDataUrl:'https://alieasset.meishesdk.com/model/fakeface_v1.0.1.dat',
                faceCommonDataUrl: "https://alieasset.meishesdk.com/model/facecommon_v1.0.0.dat",
                advancedBeautyDataUrl: "https://alieasset.meishesdk.com/model/advancedbeauty_v1.0.0.dat",
                detectionMode:(this.isImageMedia ? NveHumanDetectionFeatureEnum.SemiImageMode : NveHumanDetectionFeatureEnum.VideoMode) | NveHumanDetectionFeatureEnum.MultiThread,
                workingInRealtimeMode:!this.isImageMedia,
                jszip,
                mirror:this.mirror,
            });
        },
        async enableBeautyClick() {
            this.enableBeauty = !this.enableBeauty;
            this.arSceneRenderer.enableBeauty(this.enableBeauty);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addShape() {
            await this.arSceneRenderer.setEffectList([
                {
                    url:"https://qasset.meishesdk.com/material/pu/makeup/63BD3F32-D01B-4755-92D5-0DE361E4045A/63BD3F32-D01B-4755-92D5-0DE361E4045A.3.facemesh",
                    licUrl: "",
                    intensity:1,
                },
            ]);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addMakeup() {
            await this.arSceneRenderer.setEffectList([
                {
                    url:"https://qasset.meishesdk.com/material/pu/makeup/FDFB2239-44D1-491D-85A1-7A537860118E/FDFB2239-44D1-491D-85A1-7A537860118E.1.makeup",
                    licUrl: "",
                    intensity:1,
                },
            ]);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addSegmentationImageBackground() {
            await this.arSceneRenderer.setEffectList([
                {
                    segmentationBackgroundUrl:"/static/background.jpg",
                },
            ]);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addSegmentationBlurBackground() {
            await this.arSceneRenderer.setEffectList([
                {
                    url:"/static/F31CBEBD-9F86-4AC6-87AB-1A289A62E3AE.1.videofx",
                    licUrl: "",
                },
            ]);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addAvatar() {
            await this.arSceneRenderer.setEffectList([
                {
                    url:"https://qasset.meishesdk.com/material/pu/arscene/B4B4C7EF-47D2-4CE9-BCED-B8D8534718DC/B4B4C7EF-47D2-4CE9-BCED-B8D8534718DC.1.arscene",
                    licUrl: "",
                },
            ]);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addCompoundMakeups() {
            await this.arSceneRenderer.setEffectList([
                {
                    url:"https://alieasset.meishesdk.com/model/hy/material/EDCD5A64-26BB-4CBC-A9B0-EBAA69E91467.2.zip",
                },
            ]);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addVideoFx() {
            await this.arSceneRenderer.setEffectList([
                {
                    url:"https://qasset.meishesdk.com/material/pu/videofx/DF972925-85A1-4687-939C-5C50585A825A/DF972925-85A1-4687-939C-5C50585A825A.1.videofx",
                    licUrl: "",
                    intensity:1,
                },
            ]);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addCaption() {
            if (this.caption) {
                this.externalEffectInstanceArray.splice(this.externalEffectInstanceArray.indexOf(this.caption), 1);
                this.arSceneRenderer.setExternalEffectInstanceList(this.externalEffectInstanceArray);
                this.caption.release();
            }
            this.caption = await this.arSceneRenderer.createExternalEffectInstance(
                {
                    text: "caption",
                    inPoint: 0,
                    duration: Number.MAX_SAFE_INTEGER,
                    url: "https://qasset.meishesdk.com/material/pu/caption/193DC3A6-7EF2-4739-B8D9-9368DA2FB1B3/193DC3A6-7EF2-4739-B8D9-9368DA2FB1B3.1.captionstyle",
                    licUrl: "",
                    fontFileUrl: "https://alieasset.meishesdk.com/release/material/font/164C237B-16CB-49D5-A205-1976B16046B8/164C237B-16CB-49D5-A205-1976B16046B8.ttf",
                    positionX: -0.6,
                    positionY: 0.9,
                }
            );
            this.caption.scaleCaption2(0.8);
	        this.caption.setUnderline(true);
            this.arSceneRenderer.appendExternalEffectInstance(this.caption);
            this.externalEffectInstanceArray.push(this.caption);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addModularCaption() {
            if (this.modularCaption) {
                this.externalEffectInstanceArray.splice(this.externalEffectInstanceArray.indexOf(this.modularCaption), 1);
                this.arSceneRenderer.setExternalEffectInstanceList(this.externalEffectInstanceArray);
                this.modularCaption.release();
            }
            this.modularCaption = await this.arSceneRenderer.createExternalEffectInstance(
                {
                    modular: true,
                    text: "modularCaption",
                    inPoint: 0,
                    duration: Number.MAX_SAFE_INTEGER,
                    licUrl: "",
                    captionRendererUrl: "https://qasset.meishesdk.com/material/pu/caption/262BC53C-CB55-45D4-96BD-2BD79D3C5F11/262BC53C-CB55-45D4-96BD-2BD79D3C5F11.1.captionrenderer",
			        captionRendererLicUrl: "",
                    captionContextUrl: "https://qasset.meishesdk.com/material/pu/caption/BFC751C8-D9B2-440A-A3DF-8D02841745C1/BFC751C8-D9B2-440A-A3DF-8D02841745C1.1.captioncontext",
			        captionContextLicUrl: "",
                    fontFileUrl: "https://alieasset.meishesdk.com/release/material/font/164C237B-16CB-49D5-A205-1976B16046B8/164C237B-16CB-49D5-A205-1976B16046B8.ttf",
                    position: "top",
                }
            );
            this.modularCaption.scaleCaption2(0.8);
	        this.modularCaption.setUnderline(true);
            this.arSceneRenderer.appendExternalEffectInstance(this.modularCaption);
            this.externalEffectInstanceArray.push(this.modularCaption);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addCompoundCaption() {
            if (this.compoundCaption) {
                this.externalEffectInstanceArray.splice(this.externalEffectInstanceArray.indexOf(this.compoundCaption), 1);
                this.arSceneRenderer.setExternalEffectInstanceList(this.externalEffectInstanceArray);
                this.compoundCaption.release();
            }
            this.compoundCaption = await this.arSceneRenderer.createExternalEffectInstance(
                {
                    inPoint: 0,
                    duration: Number.MAX_SAFE_INTEGER,
                    url: "https://qasset.meishesdk.com/material/pu/compoundcaption/D037A06A-FFD8-4863-A291-A2BC105F0BBC/D037A06A-FFD8-4863-A291-A2BC105F0BBC.1.compoundcaption",
                    licUrl: "",
                    position: "top-right",
                }
            );
            this.compoundCaption.scaleCaption2(0.8);
	        this.compoundCaption.setText(0, "caption0");
            this.arSceneRenderer.appendExternalEffectInstance(this.compoundCaption);
            this.externalEffectInstanceArray.push(this.compoundCaption);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async addAnimatedSticker() {
            if (this.sticker) {
                this.externalEffectInstanceArray.splice(this.externalEffectInstanceArray.indexOf(this.sticker), 1);
                this.arSceneRenderer.setExternalEffectInstanceList(this.externalEffectInstanceArray);
                this.sticker.release();
            }
            this.sticker = await this.arSceneRenderer.createExternalEffectInstance(
                {
                    inPoint: 0,
                    duration: Number.MAX_SAFE_INTEGER,
                    url: "https://qasset.meishesdk.com/material/pu/animatedsticker/42265EE4-0799-4FEA-93DA-783BE4495F95/42265EE4-0799-4FEA-93DA-783BE4495F95.1.animatedsticker",
                    licUrl: "",
                    animatedStickerAnimationUrl: "https://qasset.meishesdk.com/material/pu/animatedsticker/58A339F9-8B2F-479F-8A1D-35E4456CFA9B/58A339F9-8B2F-479F-8A1D-35E4456CFA9B.1.animatedstickeranimation",
                    animatedStickerAnimationLicUrl: "",
                    animationPeriod: 5000,
                    positionX: 0.8,
                    positionY: -0.7,
                }
            );
            this.sticker.scaleAnimatedSticker2(0.5);
            this.arSceneRenderer.appendExternalEffectInstance(this.sticker);
            this.externalEffectInstanceArray.push(this.sticker);
            if (this.isImageMedia) {
                await this.doRenderring();
            }
        },
        async doRenderring() {
            if (this.isImageMedia) {
                const renderEffectsResult = await this.arSceneRenderer.render(this.imageData, 0);               
                this.preview.getContext("2d").drawImage(renderEffectsResult.imageBitmap, 0, 0, this.preview.width, this.preview.height);
            } else {
                this.arSceneRenderer.pushMediaStream(this.mediaStream);
                this.outputStream = this.arSceneRenderer.getOutputStream();
                this.preview.srcObject = this.outputStream;
                this.preview.play();
            }
        },
        releaseResources() {
            if (this.mediaStream) {
                this.mediaStream.getTracks().forEach((track) => {
                    track.stop();
                });
                this.mediaStream = null;
            }
            if (this.captureInterval) {
                clearInterval(this.captureInterval);
                this.captureInterval = null;
            }
            if (this.arSceneRenderer) {
                this.arSceneRenderer.release();
                this.arSceneRenderer = null;
            }
            this.externalEffectInstanceArray.forEach (item => {
                item.release();
            });
            this.externalEffectInstanceArray = [];
        },
        async useCameraMediaStream() {
            this.releaseResources();
            this.isImageMedia = false;
            this.mirror = true;
            await this.initARSceneRenderer();
            try {
                this.mediaStream = await navigator.mediaDevices.getUserMedia({
                    audio: {
                        echoCancellation: true,
                        noiseSuppression: true
                    },
                    video: {
                        width: 960,
                        height: 540,
                        frameRate: {ideal: 30, max: 60}
                    },
                });
                const videoTracks = this.mediaStream.getVideoTracks();
                let hasCamera = false;
                videoTracks.forEach((track) => {
                    if (track.kind === 'video' && track.label !== 'Intel Virtual Camera') {
                        hasCamera = true;
                    }
                });
                if (!hasCamera) {
                    this.$message.error('There is no camera device!');
                    this.releaseResources();
                    return;
                }
                await this.doRenderring();
            } catch (e) {
                this.$message.error(e.message);
            }
        },
        async useVideoMediaStream() {
            this.releaseResources();
            this.isImageMedia = false;
            this.mirror = false;
            await this.initARSceneRenderer();
            if (!this.video) {
                this.video = document.createElement('video');
            }
            this.video.hidden = "true";
            this.video.autoplay = "true";
            this.video.crossOrigin="anonymous";
            this.video.loop = 'true';
            this.video.onplay = async ()=> {
                this.mediaStream = this.video.captureStream();
                await this.doRenderring();
            }
            this.video.src = "/static/test.mp4";
        },
        async useImageMediaStream() {
            this.releaseResources();
            this.isImageMedia = false;
            this.mirror = false;
            await this.initARSceneRenderer();
            if (!this.tempImg) {
                this.tempImg = document.createElement("img");
            }
            if (!this.tempCanvas) {
                this.tempCanvas = document.createElement('canvas');
                this.tempCanvas.style.visibility = 'hidden';
                this.tempCanvas.width = 1920;
                this.tempCanvas.height = 1080;
            }
            this.tempImg.onload = () => {
                if (this.tempCanvas.width !== this.tempImg.width || this.tempCanvas.height !== this.tempImg.height) {
                    this.tempCanvas.width = this.tempImg.width;
                    this.tempCanvas.height = this.tempImg.height;
                }
                this.tempCanvas.getContext("2d").drawImage(this.tempImg, 0, 0);
            }
            this.captureInterval = setInterval(async () => {
                this.tempImg.src = "/static/test.jpg";
            }, 1000 / 30);
            this.mediaStream = this.tempCanvas.captureStream();
            await this.doRenderring();
        },
        async useImageData() {
            this.releaseResources();
            this.isImageMedia = true;
            this.mirror = false;
            await this.initARSceneRenderer();
            if (!this.tempImg) {
                this.tempImg = document.createElement("img");
            }
            this.tempImg.onload = async () => {
                if (!this.tempCanvas) {
                    this.tempCanvas = document.createElement('canvas');
                    this.tempCanvas.style.visibility = 'hidden';
                }
                this.tempCanvas.width = this.tempImg.width;
                this.tempCanvas.height = this.tempImg.height;
                const ctx = this.tempCanvas.getContext("2d");
                ctx.drawImage(this.tempImg, 0, 0);
                this.imageData = ctx.getImageData(0, 0, this.tempCanvas.width, this.tempCanvas.height);
                await this.doRenderring();
            }
            this.tempImg.src = "/static/test.jpg";
        },
    }
}
</script>

<style lang="scss" scoped>
	.main {
		height: 100%;
        display: flex;
        flex-direction: column;
        column-gap: 30px;
	}
    #preview {
		width: 960px;
		height: 540px;
	}
    .btns {
		height: 40px;
		line-height: 40px;
		margin-top: 5px;
		background: cornflowerblue;
		text-align: center;
		span {
			display: inline-block;
			padding: 0px 10px;
			margin: 0 10px;
			border-radius: 6px;
			cursor: pointer;
			&:hover {
				background: rgb(14, 45, 218);
				color: #fff;
			}
		}
	}
    .inputs {
		height: 40px;
		margin-top: 15px;
		display: flex;
		column-gap: 15px;
		align-items: center;
		button {
			height: 40px;
			background: cornflowerblue;
			cursor: pointer;
		}
	}
	.features {
		height: 40px;
		margin-top: 15px;
		display: flex;
		column-gap: 15px;
		align-items: center;
		button {
			height: 40px;
			background: cornflowerblue;
			cursor: pointer;
		}
	}
</style>
