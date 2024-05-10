<template>
    <div id="register-div" class="form-wizard" :class="[{ vertical: isVertical }]">
        <div class="wizard-header">
            <h4 class="wizard-title green">{{ title }}</h4>
            <p class="category">{{ subtitle }}</p>
        </div>
        <div class="wizard-navigation">
            <div class="wizard-progress-with-circle" v-if="!isVertical">
                <div class="wizard-progress-bar" :style="progressBarStyle"></div>
            </div>
            <ul class="wizard-nav wizard-nav-pills" role="tablist">
                <template name="step" v-for="(tab, index) in tabs" :tab="tab" :index="index"
                    :navigate-to-tab="navigateToTab" :transition="transition">
                    <li :class="{ active: tab.active }">
                        <a href="javascript:void(0)" :class="{ disabled: !tab.checked }">
                            <div class="wizard-icon-circle md" role="tab" :tabindex="tab.checked ? 0 : ''"
                                :id="`step-${tab.tabId}`" :aria-controls="tab.tabId" :aria-disabled="tab.active"
                                :aria-selected="tab.active" :class="{
                                    checked: tab.checked,
                                }" :style="[
                                    tab.checked ? stepCheckedStyle : {},
                                    tab.validationError ? errorStyle : {},
                                ]">
                                <div v-if="tab.active" class="wizard-icon-container"
                                    :style="[{ backgroundColor: color }, tab.validationError ? errorStyle : {}]">
                                    <slot name="active-step">
                                        <span class="wizard-icon" v-if="tab.customIcon" v-html="tab.customIcon"></span>
                                        <i v-else :class="tab.icon ? tab.icon : ''" class="wizard-icon"
                                            :style="tab.checked ? iconActiveStyle : ''">
                                            {{ tab.icon ? null : index + 1 }}
                                        </i>
                                    </slot>
                                </div>
                                <slot v-else>
                                    <span class="wizard-icon" v-if="tab.customIcon" v-html="tab.customIcon"></span>
                                    <i v-else :class="tab.icon ? tab.icon : ''" class="wizard-icon"
                                        :style="tab.checked ? iconActiveStyle : ''">
                                        {{ tab.icon ? null : index + 1 }}
                                    </i>
                                </slot>
                            </div>
                            <span class="stepTitle" :class="{ active: tab.active, has_error: tab.validationError }"
                                :style="tab.active || tab.checked ? stepTitleStyle(tab) : {}" style="margin-top: 5px">
                                {{ tab.title }}
                            </span>
                            <slot name="customIcon"> </slot>
                        </a>
                    </li>
                </template>
            </ul>
            <div class="wizard-tab-content">
                <div v-show="tabs[0].active" class="wizard-tab-container" role="tabpanel" :id="0"
                    :aria-hidden="!tabs[0].active" :aria-labelledby="`step-${0}`">
                    <span>Username:</span>

                    <input type="text" v-model="formData.Username" class="app-input p-2" id="username"
                        :class="{ 'app-input-error': validationRules.username }" @keyup="validateUserName()"
                        placeholder="User name" required />
                    <span v-if="validationRules.username" class="text-error">{{ validationRules.username }}</span>
                </div>
                <div v-show="tabs[1].active" class="wizard-tab-container" role="tabpanel" :id="1"
                    :aria-hidden="!tabs[1].active" :aria-labelledby="`step-${1}`">
                    <span>Email:</span>
                    <input type="text" v-model="formData.Email" class="app-input p-2" id="email"
                        :class="{ 'app-input-error': formData.Email }" placeholder="Email" required
                        @blur="validateEmail" />
                    <span class="text-error" v-if="validationRules.email">{{ validationRules.email }}</span>

                </div>
                <div v-show="tabs[2].active" class="wizard-tab-container" role="tabpanel" :id="2"
                    :aria-hidden="!tabs[2].active" :aria-labelledby="`step-${2}`">
                    <div>
                        <span>Step: review</span>
                    </div>
                    <div>
                        <span> Username: {{ formData.Username }}</span>
                    </div>
                    <div>
                        <span> Email: {{ formData.Email }}</span>
                    </div>
                </div>
            </div>
        </div>

        <div class="wizard-card-footer clearfix">
            <div class="wizard-footer-left">
                <span @click="prevTab" @keyup.enter="prevTab" role="button" tabindex="0">
                    <button class="wizard-btn" id="btn-prev" tabindex="-1" :style="fillButtonStyle"
                        :disabled="isDisablePrevButton" type="button">
                        {{ 'Back' }}
                    </button>
                </span>
            </div>

            <div class="wizard-footer-right">

                <span @click="nextTab" @keyup.enter="nextTab" role="button" tabindex="0">
                    <button class="wizard-btn" id="btn-next" :disabled="isDisableNextButton" tabindex="-1"
                        :style="fillButtonStyle" type="button">
                        {{ 'Next' }}
                    </button>
                </span>
            </div>
        </div>
    </div>
</template>
<script>



export default {

    props: {
        validateOnBack: Boolean,

        color: {
            type: String,
            default: "#1787bd",
        },
        errorColor: {
            type: String,
            default: "#8b0000",
        },

        layout: {
            type: String,
            default: "horizontal",
        },



        transition: {
            type: String,
            default: "",
        },

        startIndex: {
            type: Number,
            default: 0,
            validator: (value) => {
                return value >= 0;
            },
        },


    },

    data() {
        return {
            activeTabIndex: 0,
            currentPercentage: 0,
            maxStep: 0,
            loading: false,
            title: "Registration form",

            subtitle: "Based on the step, you should enter the specified value",

            tabs: [
                { title: 'username', active: false, checked: false },
                { title: 'email', active: false, checked: false },
                { title: 'review', active: false, checked: false },
            ],
            formData: {
                Username: "",
                Email: "",
                review: false
            },
            validationRules: []
        };
    },
    computed: {

        tabCount() {
            return this.tabs.length;
        },
        isLastStep() {
            return this.activeTabIndex === this.tabCount - 1;
        },
        isVertical() {
            return this.layout === "vertical";
        },
        displayPrevButton() {
            return this.activeTabIndex !== 0;
        },
        stepPercentage() {
            return (1 / (this.tabCount * 2)) * 100;
        },
        progressBarStyle() {
            return {
                backgroundColor: this.color,
                width: `${this.progress}%`,
                color: this.color,
            };
        },
        fillButtonStyle() {
            return {
                backgroundColor: this.color,
                borderColor: this.color,
                color: "white",
            };
        },
        progress() {
            let percentage = 0;
            if (this.activeTabIndex > 0) {
                let stepsToAdd = 1;
                let stepMultiplier = 2;
                percentage =
                    this.stepPercentage *
                    (this.activeTabIndex * stepMultiplier + stepsToAdd);
            } else {
                percentage = this.stepPercentage;
            }
            return percentage;
        },
        isDisablePrevButton() {
            switch (this.activeTabIndex) {
                case 0:
                    return true;

                default:
                    return false;
            }

        },
        isDisableNextButton() {
            switch (this.activeTabIndex) {
                case 2:
                    return true;

                default:
                    return false;
            }

        },


    },
    methods: {
        iconActiveStyleBg(color) {
            return {
                backgroundColor: color,
            };
        },
        iconActiveStyle(tab) {
            if (!tab.active) {
                return { color: tab.color };
            }
        },
        stepCheckedStyle(tab) {
            return {
                borderColor: tab.color,
            };
        },
        errorStyle(tab) {
            return {
                borderColor: tab.errorColor,
                backgroundColor: tab.errorColor,
            };
        },
        validateEmail() {
            let isvalid = false;
            if (!(/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,4})+$/.test(this.formData.Email))) {
                this.validationRules['email'] = 'Invalid email address.';
            } else {
                this.validationRules['email'] = '';
                isvalid = true;
            }
            return isvalid;
        },
        validateUserName() {
            let isvalid = false;
            if (!(/^[a-zA-Z0-9.\-_$@*!]{4,30}$/.test(this.formData.Username))) {
                this.validationRules['username'] = 'Invalid Username.';
            } else {
                this.validationRules['username'] = '';
                isvalid = true;
            }

            return isvalid;
        },
        stepTitleStyle(tab) {
            let isError = false;
            switch (tab.title) {
                case 'username':
                    isError = this.formData.Username.length <= 0;
                    break;

                default:
                    isError = this.formData.Username.length <= 0;
                    break;
            }
            return {
                color: isError ? tab.errorColor : tab.color,
            };
        },


        isPromise(func) {
            return func.then && typeof func.then === 'function'
        },

        reset() {
            this.maxStep = 0;
            this.tabs.forEach((tab) => {
                tab.checked = false;
            });
            this.navigateToTab(0);
        },

        navigateToTab(index) {
            let validate = index > this.activeTabIndex;
            if (index <= this.maxStep) {
                let cb = () => {
                    if (validate && index - this.activeTabIndex > 1) {
                        // validate all steps recursively until destination index
                        this.changeTab(this.activeTabIndex, this.activeTabIndex + 1);
                        this.beforeTabChange(this.activeTabIndex, cb);
                    } else {
                        this.changeTab(this.activeTabIndex, index);
                        this.afterTabChange(this.activeTabIndex);
                    }
                };
                if (validate) {
                    this.beforeTabChange(this.activeTabIndex, cb);
                } else {
                    this.setValidationError(null);
                    cb();
                }
            }
            return index <= this.maxStep;
        },
        checkNextValidation() {
            switch (this.activeTabIndex) {
                case 0:
                    return this.validateUserName();
                case 1:

                    return this.validateEmail();

                default:
                    return false;
            }

        },

        nextTab() {

            if (!this.checkNextValidation())
                return;

            let cb = () => {
                if (this.activeTabIndex < this.tabCount - 1) {
                    this.changeTab(this.activeTabIndex, this.activeTabIndex + 1);
                    this.afterTabChange(this.activeTabIndex);
                } else {
                    this.$emit("on-complete");
                }
            };
            this.beforeTabChange(this.activeTabIndex, cb);
        },
        prevTab() {
            let cb = () => {
                if (this.activeTabIndex > 0) {
                    this.setValidationError(null);
                    this.changeTab(this.activeTabIndex, this.activeTabIndex - 1);
                }
            };
            if (this.validateOnBack) {
                this.beforeTabChange(this.activeTabIndex, cb);
            } else {
                cb();
            }
        },

        setLoading(value) {
            this.loading = value;
            // this.$emit("on-loading", value);
        },
        setValidationError(error) {
            this.tabs[this.activeTabIndex].validationError = error;
            // this.$emit("on-error", error);
        },
        validateBeforeChange(promiseFn, callback) {
            this.setValidationError(null);
            // we have a promise
            if (isPromise(promiseFn)) {
                this.setLoading(true);
                promiseFn
                    .then((res) => {
                        this.setLoading(false);
                        let validationResult = res === true;
                        this.executeBeforeChange(validationResult, callback);
                    })
                    .catch((error) => {
                        this.setLoading(false);
                        this.setValidationError(error);
                    });
                // we have a simple function
            } else {
                let validationResult = promiseFn === true;
                this.executeBeforeChange(validationResult, callback);
            }
        },
        executeBeforeChange(validationResult, callback) {
            if (validationResult) {
                callback();
            } else {
                this.tabs[this.activeTabIndex].validationError = "error";
            }
        },
        beforeTabChange(index, callback) {
            if (this.loading) {
                return;
            }
            let oldTab = this.tabs[index];
            if (oldTab && oldTab.beforeChange !== undefined) {
                let tabChangeRes = oldTab.beforeChange();
                this.validateBeforeChange(tabChangeRes, callback);
            } else {
                callback();
            }
        },
        afterTabChange(index) {
            if (this.loading) {
                return;
            }
            let newTab = this.tabs[index];
            if (newTab && newTab.afterChange !== undefined) {
                newTab.afterChange();
            }
        },
        changeTab(oldIndex, newIndex, emitChangeEvent = true) {
            let oldTab = this.tabs[oldIndex];
            let newTab = this.tabs[newIndex];
            if (oldTab) {
                oldTab.active = false;
            }
            if (newTab) {
                newTab.active = true;
            }

            this.activeTabIndex = newIndex;
            this.activateTabAndCheckStep(this.activeTabIndex);
            return true;
        },


        deactivateTabs() {
            this.tabs.forEach((tab) => {
                tab.active = false;
            });
        },
        activateTab(index) {
            this.deactivateTabs();
            let tab = this.tabs[index];
            if (tab) {
                tab.active = true;
                // tab.checked = true;

            }
        },
        activateTabAndCheckStep(index) {
            if (index > this.maxStep) {
                this.maxStep = index;
            }
            this.activeTabIndex = index;
        },
        initializeTabs() {
            if (this.tabs.length > 0 && this.startIndex === 0) {
                this.activateTab(this.activeTabIndex);
            }
            if (this.startIndex < this.tabs.length) {
                this.activateTabAndCheckStep(this.startIndex);
            } else {
                window.console.warn(
                    `Prop startIndex set to ${this.startIndex} is greater than the number of tabs - ${this.tabs.length}. Make sure that the starting index is less than the number of tabs registered`
                );
            }
        },
    },
    mounted() {
        this.initializeTabs();
    },
    watch: {

    },
};
</script>
<style scoped lang="scss">
$gray-base: #000 !default;
$gray-darker: lighten($gray-base, 13.5%) !default; // #222
$gray-dark: lighten($gray-base, 20%) !default; // #333
$gray: lighten($gray-base, 33.5%) !default; // #555
$gray-light: lighten($gray-base, 46.7%) !default; // #777
$gray-lighter: lighten($gray-base, 93.5%) !default; // #eee

$brand-primary: darken(#428bca, 6.5%) !default; // #337ab7
$brand-success: #5cb85c !default;
$brand-info: #5bc0de !default;
$brand-warning: #f0ad4e !default;
$brand-danger: #d9534f !default;

$danger-color: #EB5E28 !default;
/////////////////////////////////////////////
$nav-disabled-link-color: $gray-light !default;
$nav-disabled-link-hover-color: $gray-light !default;
//=== Shared nav styles

$nav-link-padding: 10px 15px !default;
$nav-link-hover-bg: $gray-lighter !default;

///////////////////////////////////////////

//** Default progress bar color
$progress-bar-bg: $brand-primary !default;
//** Success progress bar color
$progress-bar-success-bg: $brand-success !default;
//** Warning progress bar color
$progress-bar-warning-bg: $brand-warning !default;
//** Danger progress bar color
$progress-bar-danger-bg: $brand-danger !default;
//** Info progress bar color
$progress-bar-info-bg: $brand-info !default;


$padding-base-vertical: 6px !default;
$padding-base-horizontal: 12px !default;

$padding-large-vertical: 10px !default;
$padding-large-horizontal: 16px !default;

$padding-small-vertical: 5px !default;
$padding-small-horizontal: 10px !default;

$padding-xs-vertical: 1px !default;
$padding-xs-horizontal: 5px !default;

/////////////////////////////

$font-size-base: 14px !default;
$font-size-large: ceil(($font-size-base * 1.25)) !default; // ~18px
$font-size-small: ceil(($font-size-base * 0.85)) !default; // ~12px

$font-size-h1: floor(($font-size-base * 2.6)) !default; // ~36px
$font-size-h2: floor(($font-size-base * 2.15)) !default; // ~30px
$font-size-h3: ceil(($font-size-base * 1.7)) !default; // ~24px
$font-size-h4: ceil(($font-size-base * 1.25)) !default; // ~18px
$font-size-h5: $font-size-base !default;
$font-size-h6: ceil(($font-size-base * 0.85)) !default; // ~12px

$border-radius-base: 4px !default;
$border-radius-large: 6px !default;
$border-radius-small: 3px !default;

$line-height-base: 1.428571429 !default;
$line-height-computed: floor(($font-size-base * $line-height-base)) !default; // ~20px

$danger-states-color: darken($danger-color, 11%) !default;

/// mixins /////////////
@mixin opacity($opacity) {
    opacity: $opacity;
    // IE8 filter
    $opacity-ie: (
        $opacity * 100
    );
filter: #{alpha(opacity=$opacity-ie)};
}

@mixin box-shadow($shadow...) {

    -webkit-box-shadow: $shadow;
    box-shadow: $shadow;
}

@mixin transition($transition...) {
    -webkit-transition: $transition;
    -o-transition: $transition;
    transition: $transition;
}

/// //////////////////////////
.form-wizard .wizard-btn {
    display: inline-block;
    margin-bottom: 0;
    font-weight: normal;
    text-align: center;
    vertical-align: middle;
    touch-action: manipulation;
    cursor: pointer;
    background-image: none;
    border: 1px solid transparent;
    white-space: nowrap;


    padding: $padding-base-vertical $padding-base-horizontal;
    font-size: $font-size-base;
    line-height: $line-height-base;
    border-radius: $border-radius-base;


    &.disabled,
    &[disabled],
    fieldset[disabled] & {
        cursor: not-allowed;
        @include opacity(.65);
        -webkit-box-shadow: none;
        box-shadow: none;
    }

}

.form-wizard * {
    box-sizing: border-box;
}

.form-wizard a {
    text-decoration: none;
}

.form-wizard .wizard-nav {
    margin-bottom: 0;
    padding-left: 0;
    list-style: none;


    >li {
        position: relative;
        display: block;

        >a {
            position: relative;
            display: block;
            padding: $nav-link-padding;

            &:hover,
            &:focus {
                text-decoration: none;
                background-color: $nav-link-hover-bg;
            }
        }

        // Disabled state sets text to gray and nukes hover/tab effects
        &.disabled>a {
            color: $nav-disabled-link-color;

            &:hover,
            &:focus {
                color: $nav-disabled-link-hover-color;
                text-decoration: none;
                background-color: transparent;
                cursor: not-allowed;
            }
        }
    }
}



// Bar of progress
.form-wizard .wizard-progress-bar {
    float: left;
    width: 0%;
    height: 100%;
    font-size: $font-size-small;
    line-height: $line-height-computed;
    color: #fff;
    text-align: center;
    background-color: $brand-primary;
    @include box-shadow(inset 0 -1px 0 rgba(0, 0, 0, .15));
    @include transition(width .6s ease);
}

//
.form-wizard .wizard-btn,
.form-wizard .navbar .navbar-nav>li>a.wizard-btn {
    box-sizing: border-box;
    border-width: 2px;
    background-color: transparent;
    font-size: $font-size-base;
    font-weight: bold;
    padding: $padding-base-vertical $padding-base-horizontal;
    min-width: 140px;

    &:hover,
    &:focus {
        outline: 0 !important;
    }
}

/// // // 



.form-wizard .wizard-nav-pills {
    margin-top: 0;
    position: relative;
    text-align: center;
    display: flex;
    flex-wrap: wrap;

    li,
    a {
        flex: 1;
        align-items: center;
        flex-wrap: wrap;
        flex-grow: 1;
    }

    a {
        display: flex;
    }

    >li>a {
        display: flex;
        flex-direction: column;
        padding: 0;
        margin: 0 auto;
        //color: $brand-primary;
        position: relative;
        top: 3px;

        &:hover,
        &:focus {
            background-color: transparent;
            color: #f0ad4e;
            outline: 0 !important;
        }

        &.disabled {
            pointer-events: none;
            cursor: default;
        }
    }

    >li.active>a,
    >li.active>a:hover,
    >li.active>a:focus {
        background-color: transparent;
        -webkit-transition: font-size .2s linear;
        -moz-transition: font-size .2s linear;
        -o-transition: font-size .2s linear;
        -ms-transition: font-size .2s linear;
        transition: font-size .2s linear;

        .wizard-icon {
            color: #FFFFFF;
            font-size: 24px;
            display: flex;
            align-items: center;
            justify-content: center;

            -webkit-transition: all .2s linear;
            -moz-transition: all .2s linear;
            -o-transition: all .2s linear;
            -ms-transition: all .2s linear;
            transition: all .2s linear;
        }
    }
}

/// wizard card

@mixin wizard-size($name, $size, $font-size) {
    $computed-font-size: calc($size/2) + 5px;

    &.#{$name} {
        .wizard-icon-circle {
            width: $size;
            height: $size;
            font-size: $font-size;


        }

        .wizard-nav-pills>li.active>a .wizard-icon {
            font-size: $font-size;
        }

        .wizard-navigation .wizard-progress-with-circle {
            position: relative;
            top: $computed-font-size;
            height: 4px;
        }
    }
}

.form-wizard {
    padding-bottom: 20px;

    .is_error {
        border-color: $danger-states-color !important;

        .icon-container {
            background: $danger-states-color !important;
        }
    }

    @include wizard-size('xs', 40px, 16px);
    @include wizard-size('sm', 50px, 20px);
    @include wizard-size('md', 70px, 24px);
    @include wizard-size('lg', 90px, 28px);

    .wizard-icon-circle {
        font-size: 18px;
        border: 3px solid #F3F2EE;
        border-radius: 50%;
        font-weight: bold;
        width: 70px;
        height: 70px;
        background-color: #252422;
        position: relative;
        display: flex;
        justify-content: center;
        align-content: center;





        .wizard-icon-container {
            display: flex;
            justify-content: center;
            flex: 1;
            border-radius: 50%;
            margin: -3px;


        }

        .wizard-icon {
            display: flex;
            align-items: center;
            justify-content: center;
        }
    }

    .wizard-tab-content {
        min-height: 100px;
        padding: 30px 20px;
    }

    .app-input {
        position: relative;
        -webkit-box-flex: 1;
        -ms-flex: 1 1 auto;
        flex: 1 1 auto;
        width: 1%;
        margin-bottom: 0;
        display: block;
        width: 100%;
        padding: .375rem .75rem;
        font-size: 1rem;
        line-height: 1.5;
        color: #495057;
        background-color: #fff;
        background-clip: padding-box;
        border: 1px solid #ced4da;
        border-radius: .25rem;
        transition: border-color .15s ease-in-out, box-shadow .15s ease-in-out;
    }

    .app-input-error {
        border-color: #f31e17;
    }

    .text-error {

        color: #f31e17;
    }

    .wizard-header {
        padding: 15px 15px 15px 15px;
        position: relative;
        border-radius: 3px 3px 0 0;
        text-align: center;
    }

    .wizard-title {
        // coor: #FFF;
        font-weight: bold;
        margin: 0;
        text-align: center;
    }

    .category {
        font-size: 14px;
        font-weight: 400;
        color: #e8e4e4;
        margin-bottom: 0px;
        text-align: center;
    }

    .wizard-navigation {
        .wizard-progress-with-circle {
            position: relative;
            top: 40px;
            height: 4px;

            .wizard-progress-bar {
                box-shadow: none;
                -webkit-transition: width .3s ease;
                -o-transition: width .3s ease;
                transition: width .3s ease;
            }
        }
    }

    .clearfix::after {
        content: "";
        clear: both;
        display: table;
    }

    .wizard-card-footer {
        padding: 0 20px;

        .wizard-footer-left {
            float: left;
        }

        .wizard-footer-right {
            float: right;
        }
    }

    @media screen and (max-width: 350px) {
        .wizard-card-footer {
            display: flex;
            justify-content: center;
            flex-direction: column;

            .wizard-footer-left,
            .wizard-footer-right {
                float: none;
                flex: 1;
                display: flex;
                justify-content: center;
            }

            .wizard-footer-right button {
                margin-top: 10px;
            }
        }
    }

    &.vertical {
        .wizard-card-footer {
            display: block;
        }

        .wizard-nav-pills {
            flex-direction: column;
        }

        .wizard-navigation {
            display: flex;
            flex-direction: row;
        }

        .wizard-card-footer {
            padding-top: 30px;

        }
    }
}
</style>