<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import api from '../services/api';

const router = useRouter();
const currentStep = ref(0);
const loading = ref(false);

/* --- Form Data --- */
const form = ref({
    familyName: '',
    message: '',
    youtubeLink: '',
    photos: [],
    wishes: [] // Default wishes will be sent
});

/* --- Steps Configuration --- */
const steps = [
    {
        id: 'intro',
        message: 'Para começar a criar esse presente especial, me conta... qual o sobrenome da sua família?',
        inputType: 'text',
        placeholder: 'Ex: Silva',
        field: 'familyName',
        buttonText: 'Continuar'
    },
    {
        id: 'message',
        message: 'Que lindo! Agora, escreva uma mensagem de Natal para aparecer no início do site.',
        inputType: 'textarea',
        placeholder: 'Desejamos a todos um Feliz Natal...',
        field: 'message',
        buttonText: 'Continuar'
    },
    {
        id: 'music',
        message: 'Você quer adicionar uma música especial? Se sim, cole o link do YouTube aqui. (Opcional)',
        inputType: 'text',
        placeholder: 'Cole o link do YouTube aqui (ou deixe em branco)',
        field: 'youtubeLink',
        buttonText: 'Continuar'
    },
    {
        id: 'photos',
        message: 'Perfeito. Agora escolha quantas fotos você quiser para colocar no albúm da sua família ',
        inputType: 'file',
        field: 'photos',
        buttonText: 'Continuar e Finalizar'
    },
    {
        id: 'payment',
        message: 'Tudo pronto! O valor para gerar o site é R$ 14,90. Escolha como pagar:',
        inputType: 'payment',
        field: null,
        buttonText: null // Button handled by payment brick
    }
];

const currentStepData = computed(() => steps[currentStep.value]);
const progress = computed(() => ((currentStep.value + 1) / steps.length) * 100);

/* --- Payment Logic --- */
const mpPublicKey = import.meta.env.VITE_MP_PUBLIC_KEY || 'TEST-00000000-0000-0000-0000-000000000000';
let brickBuilder = null;
const pixData = ref(null);

const initBrick = async () => {
  if (!window.MercadoPago) return;
  const mp = new window.MercadoPago(mpPublicKey, { locale: 'pt-BR' });
  brickBuilder = mp.bricks();
  
  const settings = {
      initialization: {
        amount: 29.90,
        payer: { email: 'test@test.com' },
      },
      customization: {
        paymentMethods: {
          ticket: "all",
          bankTransfer: "all",
          creditCard: "all",
          debitCard: "all",
          mercadoPago: "all",
        },
        visual: {
            style: { theme: 'bootstrap' }
        }
      },
      callbacks: {
        onSubmit: async (cardFormData) => {
           try {
             loading.value = true;
             const paymentResponse = await api.post('/payment', cardFormData);
             const responseData = paymentResponse.data;

             if (responseData.status === 'approved') {
                 await saveFamilyAndRedirect(responseData.id);
             } else if (responseData.status === 'pending') {
                 if (responseData.qr_code && responseData.qr_code_base64) {
                    pixData.value = { 
                        qrCode: responseData.qr_code, 
                        qrCodeBase64: responseData.qr_code_base64,
                        paymentId: responseData.id 
                    };
                 } else {
                    alert("Pagamento pendente. Verifique seu email.");
                 }
             }
           } catch (error) {
             console.error(error);
             alert("Erro no pagamento.");
           } finally {
             loading.value = false;
           }
        },
        onError: (error) => console.error(error),
      },
    };
    window.paymentBrickController = await brickBuilder.create("payment", "paymentBrick_container", settings);
};

const saveFamilyAndRedirect = async (paymentId) => {
    const formData = new FormData();
    formData.append('name', form.value.familyName);
    formData.append('message', form.value.message);
    formData.append('youtubeLink', form.value.youtubeLink);
    formData.append('paymentId', paymentId);
    
    const defaultWishes = [{ id: 1, title: 'Amor', text: 'Muito amor!', hasNewImage: false }];
    formData.append('wishes', JSON.stringify(defaultWishes));

    form.value.photos.forEach(file => {
        formData.append('photos', file);
    });

    try {
        const response = await api.post('/family', formData, {
            headers: { 'Content-Type': 'multipart/form-data' }
        });
        router.push(`/sucesso/${response.data.slug}`);
    } catch (e) {
        alert("Erro ao criar o site.");
    }
};

const checkPixStatus = async () => {
    if (pixData.value) {
        await saveFamilyAndRedirect(pixData.value.paymentId);
    }
};

/* --- Navigation & Inputs --- */
const handleNext = () => {
    if (currentStepData.value.inputType === 'text' && !form.value.familyName) return;
    if (currentStepData.value.inputType === 'textarea' && !form.value.message) return;
    if (currentStepData.value.inputType === 'file' && form.value.photos.length === 0) return;

    if (currentStep.value < steps.length - 1) {
        currentStep.value++;
        if (steps[currentStep.value].inputType === 'payment') {
            setTimeout(initBrick, 100);
        }
    }
};

const handleFiles = (event) => {
    form.value.photos = Array.from(event.target.files);
};

const handleBack = () => {
    if (currentStep.value > 0) currentStep.value--;
};
</script>

<template>
<div class="min-h-screen bg-background flex flex-col font-manrope">
    
    <!-- Top Progress Bar -->
    <div class="fixed top-0 left-0 w-full h-2 bg-gray-100 z-50">
        <div class="h-full bg-accent transition-all duration-500 ease-out" :style="{ width: progress + '%' }"></div>
    </div>

    <!-- Back Button -->
    <div class="fixed top-6 left-6 z-40">
        <button v-if="currentStep > 0" @click="handleBack" class="text-text-light hover:text-accent transition-colors flex items-center gap-2 text-sm font-bold">
            ← Voltar
        </button>
        <router-link v-else to="/" class="text-text-light hover:text-accent transition-colors flex items-center gap-2 text-sm font-bold">
            ← Ver Início
        </router-link>
    </div>

    <!-- Main Content Wizard -->
    <div class="flex-1 flex flex-col items-center justify-center p-6 w-full max-w-3xl mx-auto mt-8 mb-20">
        
        <!-- Character & Bubble -->
        <div class="flex flex-col items-center w-full mb-12 animate-fade-in-up">
            <!-- Avatar -->
            <div class="w-24 h-24 rounded-full bg-accent/10 border-4 border-white shadow-lg flex items-center justify-center relative z-10 mb-[-20px]">
                <span class="text-4xl filter drop-shadow-md">🎅</span>
            </div>
            
            <!-- Bubble -->
            <div class="bg-white px-8 py-6 rounded-2xl shadow-sm border border-border w-full text-center relative pt-10">
                <p class="text-lg md:text-xl text-text-primary font-medium leading-relaxed">
                    {{ currentStepData.message }}
                </p>
            </div>
        </div>

        <!-- Dynamic Inputs -->
        <div class="w-full animate-fade-in-up delay-100">
            
            <!-- Text Input -->
            <div v-if="currentStepData.inputType === 'text'" class="bg-white p-2 rounded-xl border border-border shadow-sm flex items-center">
                <div class="pl-4 text-accent text-xl">
                    ➜
                </div>
                <input 
                    v-model="form[currentStepData.field]"
                    type="text" 
                    :placeholder="currentStepData.placeholder"
                    @keyup.enter="handleNext"
                    class="w-full p-4 bg-transparent outline-none text-lg text-text-primary placeholder:text-gray-300"
                    autoFocus
                >
            </div>

            <!-- Textarea -->
            <div v-if="currentStepData.inputType === 'textarea'" class="bg-white p-2 rounded-xl border border-border shadow-sm">
                 <div class="pl-4 pt-2 text-accent text-xl mb-[-20px]">
                    ➜
                </div>
                <textarea 
                    v-model="form.message"
                    :placeholder="currentStepData.placeholder"
                    rows="4"
                    class="w-full p-4 pl-12 bg-transparent outline-none text-lg text-text-primary placeholder:text-gray-300 resize-none"
                    autoFocus
                ></textarea>
            </div>

            <!-- File Input -->
            <div v-if="currentStepData.inputType === 'file'" class="bg-white p-8 rounded-xl border-2 border-dashed border-border hover:border-accent transition-colors cursor-pointer group text-center relative">
                 <input type="file" multiple accept="image/*" @change="handleFiles" class="absolute inset-0 opacity-0 cursor-pointer w-full h-full z-10">
                 <div class="text-5xl mb-4 opacity-50 group-hover:opacity-100 transition-opacity">📸</div>
                 <p class="font-bold text-text-primary mb-1">Clique para enviar fotos</p>
                 <p class="text-sm text-text-light" v-if="form.photos.length === 0">Selecione suas melhores lembranças</p>
                 <p class="text-sm text-accent font-bold" v-else>{{ form.photos.length }} fotos selecionadas!</p>
            </div>

            <!-- Payment Brick -->
            <div v-show="currentStepData.inputType === 'payment'" class="bg-white p-6 rounded-xl border border-border shadow-sm">
                <div id="paymentBrick_container"></div>
            </div>

            <!-- Pix Result -->
            <div v-if="pixData" class="mt-6 bg-white p-6 rounded-xl border-2 border-accent/20 text-center animate-fade-in">
                <h3 class="font-bold text-accent mb-4">Pagamento via Pix Gerado!</h3>
                <img :src="`data:image/png;base64,${pixData.qrCodeBase64}`" class="mx-auto max-w-[200px] mb-4 rounded-lg border" />
                <textarea readonly :value="pixData.qrCode" class="w-full text-xs p-3 bg-gray-50 border rounded resize-none mb-4" rows="3"></textarea>
                <button @click="checkPixStatus" class="w-full py-3 bg-accent text-white font-bold rounded-lg hover:bg-accent/90">
                    Já realizei o pagamento
                </button>
            </div>

        </div>

    </div>

    <!-- Bottom Bar (Fixed) -->
    <div v-if="currentStepData.buttonText && !pixData" class="fixed bottom-0 left-0 w-full bg-white border-t border-border p-4 flex justify-end z-50">
        <button 
            @click="handleNext"
            class="bg-accent hover:bg-accent/90 text-white px-8 py-3 rounded-full font-bold shadow-lg transform transition-all active:scale-95 flex items-center gap-2"
        >
            {{ currentStepData.buttonText }}
            <span>➜</span>
        </button>
    </div>

</div>
</template>

<style scoped>
.animate-fade-in-up {
    animation: fadeInUp 0.5s ease-out forwards;
}
.delay-100 {
    animation-delay: 0.1s;
}

@keyframes fadeInUp {
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}
</style>
