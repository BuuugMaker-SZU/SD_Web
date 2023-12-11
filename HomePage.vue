<template>
    <el-container style="height: 550px; border: 1px solid #616161; background-color: #121212ec">

        <el-header style="height: 55px">
            <span style="text-align: left;font-size: 17px">SDPlus</span>
            <button type="text" @click="Login" class="login-button">登录</button>
        </el-header>


        <el-container style="height: 495px;">
            <el-aside style="width:415px; border: 1px solid #616161; background-color: #030303; overflow-y: auto">
                <div style="border:#616161 solid 1px;background-color: rgb(21, 21, 21); height: 50px;">
                    <el-button icon="el-icon-edit" class="custom-button"
                        :class="{ 'custom-button-clicked': WisButtonClicked }" @click="WtoggleButton">文生图</el-button>
                    <el-button icon="el-icon-picture-outline" class="custom-button"
                        :class="{ 'custom-button-clicked': TisButtonClicked }" @click="TtoggleButton">图生图</el-button>
                    <el-button icon="el-icon-pie-chart" class="custom-button"
                        :class="{ 'custom-button-clicked': HisButtonClicked }" @click="HtoggleButton">历史记录</el-button>
                </div>
                <div v-if="currentPage === '文生图页面'">
                    <el-input class="prompt" type="textarea" :autosize="false" rows="3" placeholder="正向提示词"
                        v-model="txt2img_paload.prompt" clearable></el-input>
                    <el-input class="prompt" type="textarea" :autosize="false" rows="3" placeholder="反向提示词"
                        v-model="txt2img_paload.negative_prompt" clearable></el-input>
                    <div class="sampling-container">
                        <el-button icon="el-icon-magic-stick" class="sampler">风格类型</el-button>
                        <div class="custom-select">
                            <div class="custom-select-trigger" @click="toggleOptions">
                                <span style="width: 260px;">{{ selectedOption.label }}</span>
                                <span> <i class="el-icon-arrow-down"
                                        style="font-size: 11px;margin-right: 2px;font-weight: 700;"></i></span>
                            </div>
                            <div class="custom-options" v-if="showOptions">
                                <span class="custom-option" v-for="option in options" :key="option.value"
                                    @click="selectOption(option)">
                                    <img :src="option.image" class="option-image" style="height: 50px;width: 50px;">
                                    {{ option.label }}
                                </span>
                            </div>
                        </div>
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-brush" class="sampler">采样方法</el-button>
                        <select v-model="txt2img_paload.sampler_index" placeholder="请选择采样方法" class="sample-menu">
                            <option label="Euler a" value="Euler a">Euler a</option>
                            <option label="Euler" value="Euler">Euler</option>
                            <option label="LMS" value="LMS">LMS</option>
                            <option label="Heun" value="Heun">Heun</option>
                            <option label="DPM2" value="DPM2">DPM2</option>
                            <option label="DPM2 a" value="DPM2 a">DPM2 a</option>
                            <option label="DPM++ 2S a" value="DPM++ 2S a">DPM++ 2S a</option>
                            <option label="DPM++ 2M" value="DPM++ 2M">DPM++ 2M</option>
                            <option label="DPM++ SDE" value="DPM++ SDE">DPM++ SDE</option>
                            <option label="DPM fast" value="DPM fast">DPM fast</option>
                            <option label="DPM adaptive" value="DPM adaptive">DPM adaptive</option>
                            <option label="LMS KarrasM" value="LMS KarrasM">LMS KarrasM</option>
                            <option label="DPM2 Karras" value="DPM2 Karras">DPM2 Karras</option>
                            <option label="DPM++ 2S a Karras" value="DPM++ 2S a Karras">DPM++ 2S a Karras</option>
                            <option label="DPM++ 2M Karras" value="DPM++ 2M Karras">DPM++ 2M Karras</option>
                            <option label="DPM++ SDE Karras" value="DPM++ SDE Karras">DPM++ SDE Karras</option>
                            <option label="DDIM" value="DDIM">DDIM</option>
                            <option label="PLMS" value="PLMS">PLMS</option>
                            <option label="UniPC" value="UniPC">UniPC</option>
                        </select>
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-data-line" class="sampler">采样步数</el-button>
                        <el-slider v-model="txt2img_paload.steps" :min="minValue" :max="SmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.steps" :min="minValue" :max="SmaxValue"
                            :step="stepValue" class="digital-display" @input="updateSliderValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-notebook-1" class="sampler">提示词相关性</el-button>
                        <el-slider v-model="txt2img_paload.cfg_scale" :min="minValue" :max="TmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.cfg_scale" :min="minValue" :max="TmaxValue"
                            :step="stepValue" class="digital-display" @input="updateRelationValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-tickets" class="sampler">宽</el-button>
                        <el-slider v-model="txt2img_paload.width" :min="WminValue" :max="WmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.width" :min="WminValue" :max="WmaxValue"
                            :step="stepValue" class="digital-display" @input="updateWidthValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-tickets" class="sampler">高</el-button>
                        <el-slider v-model="txt2img_paload.height" :min="WminValue" :max="WmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.height" :min="WminValue" :max="WmaxValue"
                            :step="stepValue" class="digital-display" @input="updateHeightValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-set-up" class="sampler">每批数量</el-button>
                        <el-slider v-model="txt2img_paload.batch_size" :min="minValue" :max="PmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.batch_size" :min="minValue" :max="PmaxValue"
                            :step="stepValue" class="digital-display" @input="updateSizeValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-coordinate" class="sampler">种子</el-button>
                        <input v-model="txt2img_paload.seed" class="seed-digital-display">
                    </div>
                </div>


                <div v-else-if="currentPage === '图生图页面'">
                    <el-input class="prompt" type="textarea" :autosize="false" rows="3" placeholder="正向提示词"
                        v-model="txt2img_paload.prompt" clearable></el-input>
                    <el-input class="prompt" type="textarea" :autosize="false" rows="3" placeholder="反向提示词"
                        v-model="txt2img_paload.negative_prompt" clearable></el-input>
                    <div class="upload-demo">
                        <label for="file-input" class="upload-button">点击上传</label>
                        <input type="file" id="file-input" @change="handleFileUpload" class="file-input">
                        <img v-if="imageUrl" :src="imageUrl" alt="Uploaded Image" class="pictograph">
                    </div>
                    <div>
                        <el-button v-if="imageUrl" icon="el-icon-delete" class="clear-img"
                            @click="handleUploadAgain">清空图片</el-button>
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-magic-stick" class="sampler">风格类型</el-button>
                        <div class="custom-select">
                            <div class="custom-select-trigger" @click="toggleOptions">
                                <span style="width: 260px;">{{ selectedOption.label }}</span>
                                <span> <i class="el-icon-arrow-down"
                                        style="font-size: 11px;margin-right: 2px;font-weight: 700;"></i></span>
                            </div>
                            <div class=" custom-options" v-if="showOptions">
                                <span class="custom-option" v-for="option in options" :key="option.value"
                                    @click="selectOption(option)">
                                    <img :src="option.image" class="option-image" style="height: 50px;width: 50px;">
                                    {{ option.label }}
                                </span>
                            </div>
                        </div>
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-brush" class="sampler">采样方法</el-button>
                        <select v-model="txt2img_paload.sampler_index" placeholder="请选择采样方法" class="sample-menu">
                            <option label="Euler a" value="Euler a">Euler a</option>
                            <option label="Euler" value="Euler">Euler</option>
                            <option label="LMS" value="LMS">LMS</option>
                            <option label="Heun" value="Heun">Heun</option>
                            <option label="DPM2" value="DPM2">DPM2</option>
                            <option label="DPM2 a" value="DPM2 a">DPM2 a</option>
                            <option label="DPM++ 2S a" value="DPM++ 2S a">DPM++ 2S a</option>
                            <option label="DPM++ 2M" value="DPM++ 2M">DPM++ 2M</option>
                            <option label="DPM++ SDE" value="DPM++ SDE">DPM++ SDE</option>
                            <option label="DPM fast" value="DPM fast">DPM fast</option>
                            <option label="DPM adaptive" value="DPM adaptive">DPM adaptive</option>
                            <option label="LMS KarrasM" value="LMS KarrasM">LMS KarrasM</option>
                            <option label="DPM2 Karras" value="DPM2 Karras">DPM2 Karras</option>
                            <option label="DPM++ 2S a Karras" value="DPM++ 2S a Karras">DPM++ 2S a Karras</option>
                            <option label="DPM++ 2M Karras" value="DPM++ 2M Karras">DPM++ 2M Karras</option>
                            <option label="DPM++ SDE Karras" value="DPM++ SDE Karras">DPM++ SDE Karras</option>
                            <option label="DDIM" value="DDIM">DDIM</option>
                            <option label="PLMS" value="PLMS">PLMS</option>
                            <option label="UniPC" value="UniPC">UniPC</option>
                        </select>

                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-data-line" class="sampler">采样步数</el-button>
                        <el-slider v-model="txt2img_paload.steps" :min="minValue" :max="SmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.steps" :min="minValue" :max="SmaxValue"
                            :step="stepValue" class="digital-display" @input="updateSliderValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-notebook-1" class="sampler">提示词相关性</el-button>
                        <el-slider v-model="txt2img_paload.cfg_scale" :min="minValue" :max="TmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.cfg_scale" :min="minValue" :max="TmaxValue"
                            :step="stepValue" class="digital-display" @input="updateRelationValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-tickets" class="sampler">宽</el-button>
                        <el-slider v-model="txt2img_paload.width" :min="WminValue" :max="WmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.width" :min="WminValue" :max="WmaxValue"
                            :step="stepValue" class="digital-display" @input="updateWidthValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-tickets" class="sampler">高</el-button>
                        <el-slider v-model="txt2img_paload.height" :min="WminValue" :max="WmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.height" :min="WminValue" :max="WmaxValue"
                            :step="stepValue" class="digital-display" @input="updateHeightValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-set-up" class="sampler">每批数量</el-button>
                        <el-slider v-model="txt2img_paload.batch_size" :min="minValue" :max="PmaxValue" :step="stepValue"
                            class="sampler-slider"></el-slider>
                        <input type="number" v-model="txt2img_paload.batch_size" :min="minValue" :max="PmaxValue"
                            :step="stepValue" class="digital-display" @input="updateSizeValue($event.target.value)">
                    </div>
                    <div class="sampling-container">
                        <el-button icon="el-icon-coordinate" class="sampler">种子</el-button>
                        <input v-model="txt2img_paload.seed" class="seed-digital-display">
                    </div>
                </div>


                <div v-else-if="currentPage === '历史记录页面'">
                    <el-row class="tac">
                        <el-col :span="24">
                            <el-menu :default-active="activeItem" @select="handleItemClick" class="el-menu-vertical-demo"
                                @open="handleOpen" @close="handleClose" background-color="rgba(52, 52, 52, 0.511)"
                                text-color="#fff" active-text-color="#ffd04b">
                                <el-submenu index="1">
                                    <template slot="title">
                                        <span :class="{ 'pink-color': GactiveItem === '1' }"
                                            @click="handleGItemClick('1')">2023-8</span>
                                    </template>
                                    <el-submenu index="1-1">
                                        <template slot="title">
                                            <span :class="{ 'pink-color': FactiveItem === '1-1' }"
                                                @click="handleFItemClick('1-1')">19</span>
                                        </template>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-1-1' }" index="1-1-1"
                                            @click="handleItemClick('1-1-1')">图片1</el-menu-item>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-1-2' }" index="1-1-2"
                                            @click="handleItemClick('1-1-2')">图片2</el-menu-item>
                                    </el-submenu>
                                    <el-submenu index="1-2">
                                        <template slot="title">
                                            <span :class="{ 'pink-color': FactiveItem === '1-2' }"
                                                @click="handleFItemClick('1-2')">22</span>
                                        </template>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-2-1' }" index="1-2-1"
                                            @click="handleItemClick('1-2-1')">图片1</el-menu-item>
                                    </el-submenu>
                                    <el-submenu index="1-3">
                                        <template slot="title">
                                            <span :class="{ 'pink-color': FactiveItem === '1-3' }"
                                                @click="handleFItemClick('1-3')">25</span>
                                        </template>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-3-1' }" index="1-3-1"
                                            @click="handleItemClick('1-3-1')">图片1</el-menu-item>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-3-2' }" index="1-3-2"
                                            @click="handleItemClick('1-3-2')">图片2</el-menu-item>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-3-3' }" index="1-3-3"
                                            @click="handleItemClick('1-3-3')">图片3</el-menu-item>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-3-4' }" index="1-3-4"
                                            @click="handleItemClick('1-3-4')">图片4</el-menu-item>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-3-5' }" index="1-3-5"
                                            @click="handleItemClick('1-3-5')">图片5</el-menu-item>
                                    </el-submenu>
                                    <el-submenu index="1-4">
                                        <template slot="title">
                                            <span :class="{ 'pink-color': FactiveItem === '1-4' }"
                                                @click="handleFItemClick('1-4')">26</span>
                                        </template>
                                        <el-menu-item :class="{ 'pink-color': activeItem === '1-4-1' }" index="1-4-1"
                                            @click="handleItemClick('1-4-1')">图片1</el-menu-item>
                                    </el-submenu>
                                </el-submenu>
                                <el-submenu index="2">
                                    <template slot="title">
                                        <span :class="{ 'pink-color': GactiveItem === '2' }"
                                            @click="handleGItemClick('2')">2023-9</span>
                                    </template>
                                    <el-menu-item-group>
                                        <el-submenu index="2-1">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '2-1' }"
                                                    @click="handleFItemClick('2-1')">19</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '2-1-1' }" index="2-1-1"
                                                @click="handleItemClick('2-1-1')">图片1</el-menu-item>
                                        </el-submenu>
                                        <el-submenu index="2-2">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '2-2' }"
                                                    @click="handleFItemClick('2-2')">22</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '2-2-1' }" index="2-2-1"
                                                @click="handleItemClick('2-2-1')">图片1</el-menu-item>
                                        </el-submenu>
                                        <el-submenu index="2-3">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '2-3' }"
                                                    @click="handleFItemClick('2-3')">25</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '2-3-1' }" index="2-3-1"
                                                @click="handleItemClick('2-3-1')">图片1</el-menu-item>
                                        </el-submenu>
                                    </el-menu-item-group>
                                </el-submenu>
                                <el-submenu index="3">
                                    <template slot="title">
                                        <span :class="{ 'pink-color': GactiveItem === '3' }"
                                            @click="handleGItemClick('3')">2023-10</span>
                                    </template>
                                    <el-menu-item-group>
                                        <el-submenu index="3-1">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '3-1' }"
                                                    @click="handleFItemClick('3-1')">2</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '3-1-1' }" index="3-1-1"
                                                @click="handleItemClick('3-1-1')">图片1</el-menu-item>
                                        </el-submenu>
                                        <el-submenu index="1-2">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '3-2' }"
                                                    @click="handleFItemClick('3-2')">14</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '3-2-1' }" index="3-2-1"
                                                @click="handleItemClick('3-2-1')">图片1</el-menu-item>
                                        </el-submenu>
                                        <el-submenu index="1-3">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '3-3' }"
                                                    @click="handleFItemClick('3-3')">16</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '3-3-1' }" index="3-3-1"
                                                @click="handleItemClick('3-3-1')">图片1</el-menu-item>
                                        </el-submenu>
                                    </el-menu-item-group>
                                </el-submenu>
                                <el-submenu index="4">
                                    <template slot="title">
                                        <span :class="{ 'pink-color': GactiveItem === '4' }"
                                            @click="handleGItemClick('4')">2023-11</span>
                                    </template>
                                    <el-menu-item-group>
                                        <el-submenu index="4-1">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '4-1' }"
                                                    @click="handleFItemClick('4-1')">9</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '4-1-1' }" index="1-1-1"
                                                @click="handleItemClick('4-1-1')">图片1</el-menu-item>
                                        </el-submenu>
                                        <el-submenu index="4-2">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '4-2' }"
                                                    @click="handleFItemClick('4-2')">17</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '4-2-1' }" index="4-2-1"
                                                @click="handleItemClick('4-2-1')">图片1</el-menu-item>
                                        </el-submenu>
                                        <el-submenu index="4-3">
                                            <template slot="title">
                                                <span :class="{ 'pink-color': FactiveItem === '4-3' }"
                                                    @click="handleFItemClick('4-3')">25</span>
                                            </template>
                                            <el-menu-item :class="{ 'pink-color': activeItem === '4-3-1' }" index="4-3-1"
                                                @click="handleItemClick('4-3-1')">图片1</el-menu-item>
                                        </el-submenu>
                                    </el-menu-item-group>
                                </el-submenu>


                            </el-menu>
                        </el-col>
                    </el-row>






                </div>


            </el-aside>
            <el-main style="display: flex;justify-content: center;">
                <div v-if="currentMainPage === '生图页面'">
                    <div style="display: flex; flex-direction: column">
                        <div
                            style="margin-top: 10px; background-color: rgba(52, 52, 52, 0.511); height: 300px; width: 690px; border-radius: 5px; display: flex; flex-direction: column; justify-content: center; align-items: center;">
                            <i v-if="showIcon" class="el-icon-picture"
                                style="font-size: 30px; color: rgba(231, 231, 231, 0.418);"></i>
                            <img v-if="selectedImage" :src="selectedImage" style="height: 300px; width: 300px;" />

                        </div>
                        <div style=" background-color: rgba(52, 52, 52, 0.511); margin-top: 20px; height: 50px; width:
                                690px; border-radius: 5px;flex-direction: row;">
                            <span v-for="option in createdImages" :key="option.id">
                                <img :src="option.image" @click="selectImage(option.image)"
                                    style="height: 50px; width: 50px; margin-left: 32.2px; border-radius: 5px; cursor: pointer;" />
                            </span>
                        </div>
                    </div>
                    <div style="margin-top: 20px; display: flex; justify-content: center; color: whitesmoke;">
                        <button style="margin-right: 40px;" class="main-custom-button"
                            @click="sendDataToServer()">生成图像</button>
                        <button style="margin-right: 40px;" class="main-custom-button"
                            @click="downloadImage()">保存图像</button>
                        <button class="main-custom-button" @click="openImageEditor()">二次编辑</button>
                    </div>

                </div>

                <div v-if="currentMainPage === '历史记录页面'" style="display: flex;justify-content: center;align-items: center;">
                    <div style="position: relative;border-radius: 5px;background-color: rgba(52, 52, 52, 0.511);height: 400px;width: 690px;display: flex;justify-content: center;
  align-items: center;">
                        <img src="images/image (2).png" alt="展示图片" style="width: 400px;height: 400px;" />
                        <el-button icon="el-icon-view" class="sampler" style="position: absolute; bottom: 0; right: 0;display: flex;justify-content: center;
  align-items: center;" @click="ShowOverlay">查看图片参数</el-button>
                    </div>
                    <!-- 覆盖层 -->
                    <div v-if="showOverlay" class="overlay">
                        <!-- 这里放置覆盖层的内容 -->
                        <div
                            style="position: absolute; top: 50%; left: 80%; transform: translate(-50%, -50%); background-color:  rgba(52, 52, 52, 0.511); padding: 20px;color: whitesmoke;border-radius: 5px;">
                            <div>正向提示词： </div>
                            <textarea readonly class="history_prompt" v-model="request_parameter.prompt">
                                正向提示词：
                            </textarea>
                            <div>反向提示词： </div>
                            <textarea readonly class="history_prompt"
                                v-model="request_parameter.negative_prompt">反向提示词：</textarea>
                            <div class="image-parameter-display">风格类型：{{ showHistoryStyle }}</div>
                            <div class="image-parameter-display">采样方法：{{ request_parameter.sampler_index }}</div>
                            <div class="image-parameter-display">采样步数：{{ request_parameter.steps }}</div>
                            <div class="image-parameter-display">提示词相关性：{{ request_parameter.cfg_scale }}</div>
                            <div class="image-parameter-display">宽：{{ request_parameter.width }}</div>
                            <div class="image-parameter-display">高：{{ request_parameter.height }}</div>
                            <div class="image-parameter-display">种子：{{ request_parameter.seed }}</div>
                            <div style="display: flex;">
                                <el-button icon="el-icon-printer" @click="CopyParameters" class="sampler">一键复制</el-button>
                                <el-button icon="el-icon-circle-close" @click="CloseOverlay" class="sampler">关闭</el-button>
                            </div>
                        </div>
                    </div>
                </div>
            </el-main>
        </el-container>
    </el-container>
</template>


<script>
import axios from 'axios';
// 引入美图_图片编辑器sdk
import MTImageEditor from 'mt-image-editor-sdk';


export default {
    data() {

        return {
            txt2img_paload: {
                id: '',
                prompt: '',
                negative_prompt: '',
                style: 'none',
                sampler_index: 'Euler a',// 添加采样方法的数据绑定
                steps: 10, // 步长
                width: 256,
                height: 256,
                batch_size: 1, // 每批数量
                seed: -1,
                cfg_scale: 7, // 提示词相关性
            },
            request_parameter: {
                prompt: 'pretty woman',
                negative_prompt: 'ugly,uncoordinated',
                style: 'water_color',
                sampler_index: 'LMS',// 添加采样方法的数据绑定
                steps: 3, // 步长
                width: 128,
                height: 128,
                batch_size: 1, // 每批数量
                seed: 2394111,
                cfg_scale: 10, // 提示词相关性
            },

            WisButtonClicked: true,
            TisButtonClicked: false,
            HisButtonClicked: false,
            currentPage: '文生图页面',// 添加currentPage变量，切换文、图、历史
            currentMainPage: '生图页面', //切换生图、历史记录
            SmaxValue: 25, // 采样步数最大值
            minValue: 1, // 最小值
            maxValue: 100, // 最大值
            TmaxValue: 15, // 提示词最大值
            PmaxValue: 8, // 批数量最大值
            WminValue: 64, // 高、宽最小值
            WmaxValue: 512, // 高、宽最大值
            stepValue: 1, // 步长
            imageUrl: '',
            activeItem: '', // 历史记录导航栏保存当前选中的项
            GactiveItem: '', // 历史记录导航栏保存当前选中的祖父项
            FactiveItem: '', // 历史记录导航栏保存当前选中的父项
            showIcon: true, // 初始状态显示图标
            showHistoryStyle: '',// 历史记录中的风格类型
            selectedImage: null,
            selectedOption: { id: null, label: '无类型', value: 'none', image: '' },
            showOptions: false,
            showOverlay: false, // 图片参数覆盖层是否展示
            CreatedimageUrl: 'http://localhost:5000/test', // 图片URL
            options: [
                { id: 1, label: '无类型', value: 'none', image: 'images/none.png' },
                { id: 2, label: '色彩绚丽风', value: 'detailed', image: 'images/2023-11-04_11-04-01-0.png' },
                { id: 3, label: '水彩风', value: 'water_color', image: 'images/image (2).png' },
                { id: 4, label: '手绘漫画风', value: 'Hand_drawn', image: 'images/image (3).png' },
                { id: 5, label: '像素风', value: 'pixel', image: 'images/00014-271921466.png' },
                { id: 6, label: '复古漫画风', value: 'Retro_comic', image: 'images/2023-11-04_15-26-31-0.png' },
                { id: 7, label: '机械少女风', value: 'mechanical_girl', image: 'images/2023-11-04_16-39-01-0.png' },
                { id: 8, label: '胶片风', value: 'File', image: 'images/2023-11-04_17-19-39-0.png' },
                { id: 9, label: '赛博机甲风', value: 'CyberPunk', image: 'images/2023-11-04_17-06-49-0.png' }
            ],
            // 每批次生成的所有图
            createdImages: [
                { id: 1, label: '色彩绚丽风', value: 'detailed', image: 'images/2023-11-04_11-04-01-0.png' },
                { id: 2, label: '水彩风', value: 'water_color', image: 'images/image (2).png' },
                { id: 3, label: '手绘漫画风', value: 'Hand_drawn', image: 'images/image (3).png' },
            ],

        }
    },
    created() {
        if (this.createdImages.length > 0) {
            this.showIcon = false;
            this.selectedImage = this.createdImages[0].image;

        }
        MTImageEditor.init({
            appKey: '3YmAcaIqAlinRfk5n7lafLPJLTpB0oL3', // 必填
            title: '美图秀秀Web版', 
            fullscreen:true,
            resizeAble:true,  
        });
        this.$prompt('请输入登录id', '登录', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
        }).then(({ value }) => {
            this.txt2img_paload.id = value; // 将输入的值赋给id变量
            this.$message({
                type: 'success',
                message: '你的id是: ' + value
            });   

            axios.get('http://sd.free.idcfengye.com//i2ihistory/'+value)
                .then(response => {
                    // Handle the response from the server
                    // this.getImage();
                    console.log(response);
                })
                .catch(error => {
                    // Handle any errors that occur during the request
                    console.error(error);
                });
            axios.get('http://sd.free.idcfengye.com//t2ihistory/'+value)
                .then(response => {
                    // Handle the response from the server
                    // this.getImage();
                    console.log(response);
                })
                .catch(error => {
                    // Handle any errors that occur during the request
                    console.error(error);
                });
        }).catch(() => {
            this.$message({
                type: 'info',
                message: '取消输入'
            });
        });
    },
    methods: {
        Login() {
            this.$prompt('请输入登录id', '登录', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
            }).then(({ value }) => {
                this.txt2img_paload.id = value; // 将输入的值赋给id变量
                this.$message({
                    type: 'success',
                    message: '你的id是: ' + value
                });   
            }).catch(() => {
                this.$message({
                    type: 'info',
                    message: '取消输入'
                });
            });
        },
        WtoggleButton() {
            this.WisButtonClicked = true;
            this.TisButtonClicked = false;
            this.HisButtonClicked = false;
            this.currentMainPage = '生图页面'
            this.currentPage = '文生图页面'; // 设置currentPage为对应的页面
        },
        TtoggleButton() {
            this.WisButtonClicked = false;
            this.TisButtonClicked = true;
            this.HisButtonClicked = false;
            this.currentMainPage = '生图页面'
            this.currentPage = '图生图页面'; // 设置currentPage为对应的页面
        },
        HtoggleButton() {
            if (this.txt2img_paload.id !== '') {
                this.HisButtonClicked = true;
                this.WisButtonClicked = false;
                this.TisButtonClicked = false;
                this.currentMainPage = '历史记录页面'
                this.currentPage = '历史记录页面';
            } else {
                this.HisButtonClicked = false;
                this.$message({
                    type: 'warning',
                    message: '请先登录'
                });
            }
        },
        updateSliderValue(newValue) {
            // 通过监听 <input> 的 @input 事件，将输入框的值更新到 <el-slider> 中
            this.$nextTick(() => {
                this.steps = Number(newValue);
            });
        },
        updateRelationValue(newValue) {
            // 通过监听 <input> 的 @input 事件，将输入框的值更新到 <el-slider> 中
            this.$nextTick(() => {
                this.relation = Number(newValue);
            });
        },
        updateWidthValue(newValue) {
            // 通过监听 <input> 的 @input 事件，将输入框的值更新到 <el-slider> 中
            this.$nextTick(() => {
                this.width = Number(newValue);
            });
        },
        updateHeightValue(newValue) {
            // 通过监听 <input> 的 @input 事件，将输入框的值更新到 <el-slider> 中
            this.$nextTick(() => {
                this.height = Number(newValue);
            });
        },
        updateSizeValue(newValue) {
            // 通过监听 <input> 的 @input 事件，将输入框的值更新到 <el-slider> 中
            this.$nextTick(() => {
                this.batch_size = Number(newValue);
            });
        },
        handleFileUpload(event) {
            const file = event.target.files[0];
            this.imageUrl = URL.createObjectURL(file);
            if (file) {
                const reader = new FileReader();

                reader.onload = (e) => {
                    const fileContent = e.target.result;
                    // 将文件内容转换成 base64 编码的字符串
                    const base64Content = btoa(fileContent);

                    console.log("查看内容：" + base64Content);
                    // 将 base64 字符串包装在 JSON 对象中
                    const jsonData = { content: base64Content };

                    console.log("是不是json:" + jsonData);
                    // 将 JSON 对象发送到后端
                    this.sendToBackend(jsonData);
                };

                // 以 Data URL 形式读取文件内容
                reader.readAsDataURL(file);
            }

        },
        /* 将数据传到后端生成图片 */
        sendToBackend(jsonString) {
            // 使用 Axios 发送 POST 请求
            axios.post('http://sd.free.idcfengye.com/img2img', jsonString, {
                headers: {
                    'Content-Type': 'application/json', // 设置请求头为 JSON 类型
                },
            })
                .then(response => {
                    // 处理响应
                    console.log(response.data);
                })
                .catch(error => {
                    // 处理错误
                    console.error(error);
                });
        },

        /* 保存图像到本地 */
        downloadImage() {
            // 获取图像元素
            const image = this.$el.querySelector('img');

            // 创建一个链接
            const a = document.createElement('a');
            a.href = image.src;
            a.download = 'downloaded_image.jpg'; // 下载后的文件名

            // 模拟点击链接进行下载
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
        },
        handleUploadAgain() {
            this.imageUrl = null; // 清空图片链接
            this.$refs.fileInput.value = ''; // 重置文件选择框的值
        },
        handleOpen(key, keyPath) {
            console.log(key, keyPath);
        },
        handleClose(key, keyPath) {
            console.log(key, keyPath);
        },
        handleItemClick(index) {
            this.activeItem = index; // 更新当前选中的项
        },
        handleGItemClick(index) {
            this.GactiveItem = index; // 更新当前选中的祖父项
        },
        handleFItemClick(index) {
            this.FactiveItem = index; // 更新当前选中的父项
        },
        /* 将文生图所需数据传到后端 */
        sendDataToServer() {
            const data = this.txt2img_paload;

            axios.post('http://sd.free.idcfengye.com/genertate', data)
                .then(response => {
                    // Handle the response from the server
                    // this.getImage();
                    console.log(response);
                })
                .catch(error => {
                    // Handle any errors that occur during the request
                    console.error(error);
                });
        },
        getImage() {
            this.CreatedimageUrl = `http://sd.free.idcfengye.com/test`;
        },
        openImageEditor() {
            // 生成带随机参数的图片 URL
            const imageUrlWithRandomParam = `${this.CreatedimageUrl}`;
            console.log(imageUrlWithRandomParam)
            MTImageEditor.openImage('http://sd.free.idcfengye.com/test');
            MTImageEditor.onClose(()=>{ console.log('弹窗关闭');});
        },

        /* 选择风格类型 */
        toggleOptions() {
            this.showOptions = !this.showOptions;
        },
        selectOption(option) {
            this.selectedOption = option;
            this.selectedOption = Object.assign({}, option);
            this.showOptions = false;
            this.txt2img_paload.style = option.value;

        },
        /* 展示选中图片 */
        selectImage(imageUrl) {
            this.selectedImage = imageUrl;
        },
        /* 展示图片参数 */
        ShowOverlay() {
            this.showOverlay = true;
            // 假设 selectedValue 是你想要查找的 value
            const selectedValue = this.request_parameter.style;

            // 使用 Array.find() 方法找到对应的 option
            const selectedOption = this.options.find(option => option.value === selectedValue);

            // 确保找到了匹配的 option
            if (selectedOption) {
                // 这里可以访问到对应的 option
                this.showHistoryStyle = selectedOption.label;
            } else {
                // 如果没有找到匹配的 option
                console.log('未找到匹配的 option');
            }

        },
        CloseOverlay() {
            this.showOverlay = false;
        },
        /* 一键复制历史图片参数 */
        CopyParameters() {
            this.txt2img_paload = this.request_parameter;
            // 假设 selectedValue 是你想要查找的 value
            const selectedValue = this.request_parameter.style;

            // 使用 Array.find() 方法找到对应的 option
            const selectedOption = this.options.find(option => option.value === selectedValue);

            // 确保找到了匹配的 option
            if (selectedOption) {
                // 这里可以访问到对应的 option
                this.selectedOption = selectedOption;
            } else {
                // 如果没有找到匹配的 option
                console.log('未找到匹配的 option');
            }
            this.$alert('复制成功', '提示', {
                confirmButtonText: '确定',
            });
        }

    }
};

</script>


<style>
.el-header {
    background-color: #060606;
    color: hotpink;
    line-height: 60px;
}

.el-aside {
    color: #333;
}

/* 调整登录按钮样式 */
.login-button {
    background-color: hotpink !important;
    color: black !important;
    border-radius: 8px !important;
    border: #060606a2 solid 2px !important;
    width: 110px !important;
    height: 40px !important;
    margin-left: 980px !important;
    font-weight: 550 !important;
    font-size: large !important;
}

/* 调整布局 */
.sampling-container {
    display: flex;
    align-items: center;
}

/* 文生图、图生图、历史按钮样式 */
.custom-button {
    margin-left: 5px !important;
    margin-top: 5px !important;
    background-color: rgba(52, 52, 52, 0.511) !important;
    color: whitesmoke !important;
    border-radius: 5px !important;
    border: #060606 solid !important;
    width: 125px !important;
    height: 40px !important;
}

.custom-button-clicked {
    margin-left: 5px !important;
    margin-top: 5px !important;
    background-color: hotpink !important;
    color: black !important;
    border-radius: 5px !important;
    border: #060606 solid !important;
    width: 125px !important;
    height: 40px !important;
}

/* 提示词样式 */
.prompt {
    background-color: rgba(52, 52, 52, 0.511) !important;
    color: whitesmoke !important;
    width: 387px !important;
    margin-top: 5px !important;
    margin-left: 5px !important;
    margin-right: 5px !important;

}

/* 历史提示词样式 */
.history_prompt {
    background-color: rgba(52, 52, 52, 0.511) !important;
    color: whitesmoke !important;
    width: 200px !important;
    margin-top: 5px !important;
    margin-left: 5px !important;
    margin-right: 5px !important;

}

.prompt .el-textarea__inner {
    background-color: rgba(52, 52, 52, 0.511);
    border: #333 1px solid;
    color: whitesmoke;
    /* 设置文本区域的背景颜色为灰色 */
}

/* 风格转换 */
.custom-select {
    position: relative;
    display: inline-block;
}

.custom-select-trigger {
    position: relative;
    display: block;
    width: 274px;
    height: 40px;
    margin-left: 8px;
    margin-top: 8px;
    padding-left: 4px;
    font-size: 14px;
    font-weight: 500;
    color: #fff;
    background-color: rgba(52, 52, 52, 0.511);
    border-radius: 5px;
    cursor: pointer;
    border: #fffefe23 solid 1px;
    display: flex;
    align-items: center;
}

.custom-options {
    width: 277px;
    height: 320px;
    margin-left: 8px;
    border-radius: 5px;
    background-color: #333;
    position: absolute;
    z-index: 9999;
    top: 100%;
    left: 0;
}

.custom-select.active.custom-options {
    display: block;
}


.custom-option {
    position: absolute;
    position: relative;
    z-index: 9999;
    width: 278px;
    height: 70px;
    border-radius: 5px;
    display: block;
    line-height: 36px;
    font-size: 14px;
    color: #fff;
    background-color: #333;
    cursor: pointer;
    display: flex;
    align-items: center;

}

.custom-option:hover {
    background-color: hotpink;
}

.custom-option.selected {
    color: #333;
    background-color: white;
}

.option-image {
    width: 50px;
    height: 50px;
    margin-right: 10px;
    margin-left: 10px;
}

/* 图片样式 */
.custom-option img {
    width: 20px;
    height: 20px;
    margin-right: 10px;
}

/* 图生图上传样式 */
.upload-demo {
    border: 1px dotted #060606;
    border-radius: 5px;
    padding: 20px;
    margin-top: 8px;
    margin-left: 5px;
    width: 344px;
    height: 250px;
    position: relative;
    overflow: hidden;
    /* 超出容器大小的部分隐藏 */
    text-align: center;
    background-color: rgba(52, 52, 52, 0.511);
}

/* 图生图图片样式 */
.pictograph {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    max-width: 100%;
    max-height: 100%;
}

.file-input {
    opacity: 0;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    cursor: pointer;
}

/* 上传图片按钮样式 */
.upload-button {
    margin-top: 100px;
    display: inline-block;
    background-color: hotpink !important;
    color: black !important;
    border-radius: 5px !important;
    border: #060606 solid !important;
    width: 100px !important;
    height: 30px !important;
    cursor: pointer;
    line-height: 30px;
    text-align: center !important;
}

.upload-button:hover {
    background-color: #e0e0e0;
}

.style-sampler {
    position: relative;
    background-color: rgba(52, 52, 52, 0.511) !important;
    color: whitesmoke !important;
    border-radius: 5px !important;
    border: #fffefe23 solid 1px !important;
    margin-top: 8px !important;
    margin-left: 8px !important;
    font-size: small !important;
    width: 280px !important;
    height: 40px;
    display: flex !important;
    align-items: center !important;
    /* 垂直居中 */

}

/* 采样方法总样式 */
.sampler {
    position: relative;
    background-color: rgba(52, 52, 52, 0.511) !important;
    color: whitesmoke !important;
    border-radius: 5px !important;
    border: #fffefe23 solid 1px !important;
    margin-top: 8px !important;
    margin-left: 5px !important;
    font-size: small !important;
    width: 100px !important;
    display: flex !important;
    align-items: center !important;
    /* 垂直居中 */
    justify-content: center !important;
    /* 水平居中 */
}

.clear-img {
    position: relative;
    background-color: rgba(52, 52, 52, 0.511) !important;
    color: whitesmoke !important;
    border-radius: 5px !important;
    border: #fffefe23 solid 1px !important;
    margin-top: 8px !important;
    margin-left: 291px !important;
    font-size: small !important;
    width: 100px !important;
    display: flex !important;
    align-items: center !important;
    /* 垂直居中 */
    justify-content: center !important;
    /* 水平居中 */
}

/* 采样方法样式 */
.sample-menu {
    position: relative;
    margin-top: 8px !important;
    margin-left: 8px !important;
    width: 278px !important;
    height: 40px;
    background-color: rgba(52, 52, 52, 0.511) !important;
    border: #fffefe23 solid 1px !important;
    border-radius: 5px;
    color: whitesmoke !important;
}

/* 采样步长样式 */
.sampler-slider {
    position: relative;
    margin-top: 8px !important;
    margin-left: 8px !important;
    width: 200px !important;
    z-index: 1;
    /* 将 z-index 设置为较低的值 */
}

.sampler-slider .el-slider__runway,
.sampler-slider .el-slider__bar {
    position: relative;
    background-color: hotpink;
    /* 设置背景颜色为粉色 */
}

.sampler-slider .el-slider__button-wrapper .el-slider__button {
    position: static;
    border-color: hotpink;
    /* 设置边框颜色为粉色 */
}

/* 展示数字样式 */
.digital-display {
    position: relative;
    margin-top: 8px;
    margin-left: 8px;
    background-color: rgba(52, 52, 52, 0.511);
    color: whitesmoke;
    border-radius: 5px;
    width: 65px;
    height: 35px;
    border: #fffefe23 solid 1px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: large;
}

/* 种子展示数字样式 */
.seed-digital-display {
    position: relative;
    margin-top: 8px;
    margin-left: 8px;
    background-color: rgba(52, 52, 52, 0.511);
    color: whitesmoke;
    border-radius: 5px;
    width: 195px;
    height: 35px;
    border: #fffefe23 solid 1px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: large;
    margin-bottom: 8px;
}

/* 主页面按钮样式 */
.main-custom-button {
    background-color: hotpink !important;
    color: black !important;
    border-radius: 5px !important;
    border: #060606 solid !important;
    width: 185px !important;
    height: 50px !important;
    font-size: large;
}

/* 历史记录导航栏样式 */
.tac {
    margin-top: 5px;
}

.el-menu-item.pink-color {
    color: hotpink !important;
}

/* 查看图片参数的覆盖样式 */
.overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    /* 半透明遮罩层 */
    z-index: 999;
    /* 确保覆盖在其他内容之上 */
}

/* 图片参数展示样式 */
.image-parameter-display {
    padding-bottom: 5px;

}
</style>