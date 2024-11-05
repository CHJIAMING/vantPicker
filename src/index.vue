<template>
    <!-- 显示翻译后的文本 -->
    <van-field v-if="translatedText" v-model="translatedText" :label="label" readonly is-link @click="showPicker = true"
        label-width="80" :required="required" />
    <!-- 显示占位符 -->
    <van-field v-else :label="label" :placeholder="`请选择${label}`" readonly is-link :rules="rules"
        @click="showPicker = true" label-width="80" :required="required" />
    <!-- 弹出选择器 -->
    <van-popup v-model:show="showPicker" position="bottom">
        <van-picker v-model="selectedOption" :title="label" :columns="dictList" @confirm="onPickerConfirm"
            @cancel="showPicker = false" :loading="loadingPicker" />
    </van-popup>
</template>

<script setup>
import { ref, watch, onMounted, toRefs, computed } from "vue";
import { getDicts } from "@/api/base";

const props = defineProps({
    modelValue: {
        type: [String, Number],
        default: "",
    },
    label: {
        type: String,
        default: "",
    },
    dict: {
        type: String,
        default: "",
    },
    rules: {
        type: Object,
        default: () => ({}),
    },
    required: {
        type: Boolean,
        default: true
    }
});

const emit = defineEmits(["update:modelValue"]);
const { modelValue, dict, rules } = toRefs(props);

const dictList = ref([]); // 字典请求数据
const loadingPicker = ref(true); // 选择器loading
const showPicker = ref(false); // 选择器显示
const selectedOption = ref([]); // 当前选中的选项

// 计算属性：翻译 modelValue 对应的文本
const translatedText = computed(() => {
    const item = dictList.value.find((item) => item.value === modelValue.value);
    return item ? item.text : "";
});

// 监听 pickerVal 的变化
watch(modelValue, (val) => {
    selectedOption.value[0] = val;
});

// 获取字典数据
onMounted(async () => {
    const res = await getDicts(dict.value);
    dictList.value = res.data[0].array;
    loadingPicker.value = false;
});

// 选择器确认事件
const onPickerConfirm = ({ selectedOptions }) => {
    const selectedValue = selectedOptions[0]?.value || "";
    emit("update:modelValue", selectedValue);
    showPicker.value = false;
};
</script>