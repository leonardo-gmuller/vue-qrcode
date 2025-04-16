<!-- src/components/QrGenerator.vue -->
<template>
    <div class="grid gap-5">
        <Card>
            <template #content>
                <div>
                    <Accordion :value="['0', '1', '2']" multiple>
                        <AccordionPanel value="0">
                            <AccordionHeader>Principal</AccordionHeader>
                            <AccordionContent>
                                <div class="mt-2 grid gap-3">
                                    <div>
                                        <div>
                                            <FloatLabel variant="on">
                                                <InputText id="on_label" v-model="text" fluid
                                                    @update:modelValue="updateQRCode" />
                                                <label for="on_label">Link</label>
                                            </FloatLabel>
                                        </div>
                                        <div class="mt-3 text-left">
                                            <FloatLabel variant="on">
                                                <Select fluid v-model="dotsType" :options="optionsDot"
                                                    optionLabel="label" optionValue="value"
                                                    @update:modelValue="updateQRCode" />
                                                <label for="on_label">Estilo dos pontos</label>
                                            </FloatLabel>
                                        </div>
                                    </div>
                                    <div class="">
                                        <FileUpload customUpload @select="onFileSelect" accept="image/*"
                                            chooseLabel="Escolha uma imagem" cancelLabel="Cancelar" :fileLimit="1"
                                            @remove="onFileRemove">
                                            <template #header="{ chooseCallback, files }">
                                                <div class="flex align-items-center">
                                                    <Button label="Escolher imagem" icon="pi pi-fw pi-plus"
                                                        @click="chooseCallback" />
                                                </div>
                                            </template>
                                            <template #content="{ files, removeFileCallback, messages }">
                                                <div class="flex flex-column">
                                                    <Message v-for="message of messages" :key="message"
                                                        :class="{ 'mb-8': !files.length && !uploadedFiles.length }"
                                                        severity="error">
                                                        {{ message }}
                                                    </Message>
                                                    <div class="flex flex-column">
                                                        <div v-for="file of files" :key="file.name"
                                                            class="flex align-items-center">
                                                            <img :src="file.objectURL" alt="File"
                                                                class="w-3rem h-3rem mr-2" />
                                                            <span>{{ file.name }}</span>
                                                            <Button icon="pi pi-times" class="ml-auto"
                                                                @click="removeFileCallback(file)" rounded
                                                                severity="danger" size="small" outlined />
                                                        </div>
                                                    </div>
                                                </div>
                                            </template>
                                        </FileUpload>
                                    </div>
                                </div>
                            </AccordionContent>
                        </AccordionPanel>
                        <AccordionPanel value="1">
                            <AccordionHeader>Fundo</AccordionHeader>
                            <AccordionContent>
                                <div class="mt-2 flex align-items-center">
                                    <div>
                                        <IconField>
                                            <FloatLabel variant="on">
                                                <InputIcon class="pi pi-palette" />
                                                <InputText id="on_label" v-model="colorBackground" />
                                                <label for="on_label">Cor de fundo</label>
                                            </FloatLabel>
                                        </IconField>

                                    </div>
                                    <ColorPicker class="ml-2" v-model="colorBackground" format="hex" />
                                </div>
                            </AccordionContent>
                        </AccordionPanel>
                        <AccordionPanel value="2">
                            <AccordionHeader>Cantos</AccordionHeader>
                            <AccordionContent>
                                <div class="mt-2">
                                    <div class="text-left">
                                        <FloatLabel variant="on">
                                            <Select fluid v-model="cornerSquareType" :options="optionsCornerSquare"
                                                optionLabel="label" optionValue="value"
                                                @update:modelValue="updateQRCode"></Select>
                                            <label for="on_label">Estilo dos cantos (exterior)</label>
                                        </FloatLabel>
                                    </div>
                                    <div class="mt-3 flex align-items-center">
                                        <div>
                                            <IconField>
                                                <FloatLabel variant="on">
                                                    <InputIcon class="pi pi-palette" />
                                                    <InputText id="on_label" v-model="colorEyes" />
                                                    <label for="on_label">Cor de dentro</label>
                                                </FloatLabel>
                                            </IconField>
                                        </div>
                                        <ColorPicker class="ml-2" v-model="colorEyes" format="hex" />
                                    </div>
                                    <div class="mt-3 text-left">
                                        <FloatLabel variant="on">
                                            <Select fluid v-model="cornerDotType" :options="optionsCornerSquare"
                                                optionLabel="label" optionValue="value"
                                                @update:modelValue="updateQRCode" />
                                            <label for="on_label">Estilo dos cantos (interior)</label>
                                        </FloatLabel>
                                    </div>
                                    <div class="mt-3 flex align-items-center">
                                        <div>
                                            <IconField>
                                            <FloatLabel variant="on">
                                                <InputIcon class="pi pi-palette" />
                                                <InputText id="on_label" v-model="colorBackEyes" />
                                                <label for="on_label">Cor de fora</label>
                                            </FloatLabel>
                                        </IconField>
                                        </div>
                                        <ColorPicker class="ml-2" v-model="colorBackEyes" format="hex" />
                                    </div>
                                </div>
                            </AccordionContent>
                        </AccordionPanel>
                    </Accordion>
                    <div class="mt-4">
                        <Button label="Cancelar" :onClick="reset" severity="danger" text />
                    </div>
                </div>
            </template>
        </Card>
        <div class="flex flex-column align-items-center w-full md:w-auto lg:w-auto xl:w-auto">
            <div class="mt-auto mb-auto">
                <div ref="qrCodeRef"></div>
                <div class="mt-3 flex align-items-center justify-content-center">
                    <Button label="Salvar" :onClick="save" />
                    <div>
                        <Select class="ml-2" v-model="typeDownload" :options="typeDownloadOptions" optionLabel="label"
                            optionValue="value" />
                    </div>
                </div>

            </div>

        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import QRCodeStyling from 'qr-code-styling'
import ColorPicker from 'primevue/colorpicker';
import InputText from 'primevue/inputtext';
import FloatLabel from 'primevue/floatlabel';
import Button from 'primevue/button';
import FileUpload from 'primevue/fileupload';
import Accordion from 'primevue/accordion';
import AccordionPanel from 'primevue/accordionpanel';
import AccordionHeader from 'primevue/accordionheader';
import AccordionContent from 'primevue/accordioncontent';
import Card from 'primevue/card';
import Select from 'primevue/select';
import Message from 'primevue/message';
import IconField from 'primevue/iconfield';
import InputIcon from 'primevue/inputicon';
import InputNumber from 'primevue/inputnumber';


const text = ref("https://exemplo.com")
const qrCodeRef = ref(null)
const colorBackground = ref("#000000");
const colorEyes = ref("#000000");
const colorBackEyes = ref("#000000");
const srcimg = ref(null);
const dotsType = ref("rounded");
const cornerSquareType = ref("rounded");
const cornerDotType = ref("rounded");
const optionsCornerSquare = ref([
    { label: 'Ponto', value: 'dot' },
    { label: 'Pontos', value: 'dots' },
    { label: 'Quadrado', value: 'square' },
    { label: 'Arredondado', value: 'rounded' },
    { label: 'Super Arredondado', value: 'extra-rounded' },
    { label: 'Elegante', value: 'classy' },
    { label: 'Elegante Arredondado', value: 'classy-rounded' }
]);
const optionsDot = ref([
    { label: 'Quadrado', value: 'square' },
    { label: 'Pontos', value: 'dot' },
    { label: 'Arredondado', value: 'rounded' },
    { label: 'Super Arredondado', value: 'extra-rounded' },
    { label: 'Elegante', value: 'classy' },
    { label: 'Elegante Arredondado', value: 'classy-rounded' }
]);
const typeDownloadOptions = ref([
    { label: 'PNG', value: 'png' },
    { label: 'JPEG', value: 'jpeg' },
    { label: 'WEBP', value: 'webp' },
    { label: 'SVG', value: 'svg' },
]);
const typeDownload = ref('png');

const qrCode = new QRCodeStyling({
    width: 300,
    height: 300,
    data: text.value,
    margin: 6,
    dotsOptions: {
        color: colorBackground.value,
        type: dotsType.value
    },
    cornersSquareOptions: {
        type: cornerSquareType.value,
        color: colorEyes.value
    },
    backgroundOptions: {
        color: "#ffffff"
    },
    imageOptions: {
        crossOrigin: "anonymous",
        margin: 5
    },

})

onMounted(() => {
    qrCode.append(qrCodeRef.value)
})


const updateQRCode = () => {
    qrCode.update({
        data: text.value,
        cornersSquareOptions: {
            color: colorBackEyes.value.startsWith('#') ? colorBackEyes.value : '#' + colorBackEyes.value,
            type: cornerSquareType.value
        },
        cornersDotOptions: {
            color: colorEyes.value.startsWith('#') ? colorEyes.value : '#' + colorEyes.value,
            type: cornerDotType.value
        },
        dotsOptions: {
            color: colorBackground.value.startsWith('#') ? colorBackground.value : '#' + colorBackground.value,
            type: dotsType.value
        },
        image: srcimg.value,
    })
}

const onFileSelect = (event) => {
    const file = event.files[0];
    const reader = new FileReader();

    reader.onload = async (e) => {
        srcimg.value = e.target.result;
        updateQRCode();
    };

    reader.readAsDataURL(file);

}

const onFileRemove = (event) => {
    srcimg.value = null;
    updateQRCode();
}

const save = () => {
    const qrCodeToDownload = new QRCodeStyling({
        width: 1080,
        height: 1080,
        data: text.value,
        margin: 6,
        dotsOptions: {
            color: colorBackground.value.startsWith('#') ? colorBackground.value : '#' + colorBackground.value,
            type: dotsType.value
        },
        cornersSquareOptions: {
            type: cornerSquareType.value,
            color: colorBackEyes.value.startsWith('#') ? colorBackEyes.value : '#' + colorBackEyes.value
        },
        cornersDotOptions: {
            type: cornerDotType.value,
            color: colorEyes.value.startsWith('#') ? colorEyes.value : '#' + colorEyes.value
        },
        backgroundOptions: {
            color: "#ffffff"
        },
        imageOptions: {
            crossOrigin: "anonymous",
            margin: 5
        },
        image: srcimg.value,
    });
    qrCodeToDownload.download({
        name: "qr-code",
        extension: typeDownload.value,
    })
}

const reset = () => {
    text.value = "https://exemplo.com";
    colorBackground.value = "#000000";
    colorBackEyes.value = "#000000";
    colorEyes.value = "#000000";
    srcimg.value = null;
    dotsType.value = "rounded";
    cornerSquareType.value = "rounded";
    cornerDotType.value = "rounded";
    updateQRCode();
}

const formatColor = (color) => {
    if (!color.startsWith('#')) {
        return color = '#' + color;
    }
    return color;
}
watch(colorBackground, (newValue) => {
    colorBackground.value = formatColor(newValue);
    updateQRCode();
})
watch(colorBackEyes, (newValue) => {
    colorBackEyes.value = formatColor(newValue);
    updateQRCode();
})
watch(colorEyes, (newValue) => {
    colorEyes.value = formatColor(newValue);
    updateQRCode();
})
</script>