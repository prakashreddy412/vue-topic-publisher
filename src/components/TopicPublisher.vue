<template>
  <div class="topic-publisher-container">
    <a-card :bordered="false" class="publisher-card">
      <template #title>
        <div class="card-header">
          <span class="header-title">Create New Topic</span>
          <span class="header-subtitle">Publish your thoughts to the world</span>
        </div>
      </template>
      
      <a-form layout="vertical" class="publisher-form">
        <a-form-item label="Title">
          <a-input 
            v-model:value="formState.title" 
            placeholder="Enter an engaging title" 
            size="large"
            class="custom-input"
          />
        </a-form-item>

        <a-form-item label="Language">
          <a-select 
            v-model:value="currentLang" 
            @change="handleLanguageChange" 
            size="large"
            class="custom-select"
          >
            <a-select-option value="en-US">🇺🇸 English (US)</a-select-option>
            <a-select-option value="te-IN">🇮🇳 Telugu (తెలుగు)</a-select-option>
            <a-select-option value="hi-IN">🇮🇳 Hindi (हिन्दी)</a-select-option>
            <a-select-option value="ta-IN">🇮🇳 Tamil (தமிழ்)</a-select-option>
            <a-select-option value="kn-IN">🇮🇳 Kannada (ಕನ್ನಡ)</a-select-option>
          </a-select>
        </a-form-item>

        <a-form-item label="Description">
          <div class="description-container">
            <a-textarea
              v-model:value="formState.description"
              placeholder="Start typing or use voice..."
              :rows="6"
              class="description-input custom-textarea"
            />
            
            <div class="mic-controls">
              <a-tooltip :title="isListening ? 'Stop Recording' : 'Start Recording'">
                <a-button
                  :type="isListening ? 'primary' : 'default'"
                  :class="['mic-fab', { 'is-recording': isListening }]"
                  shape="circle"
                  size="large"
                  @click="toggleListening"
                >
                  <template #icon>
                    <div class="mic-icon-wrapper">
                      <AudioFilled v-if="!isListening" />
                      <div v-else class="waveform-icon">
                        <span></span><span></span><span></span>
                      </div>
                    </div>
                  </template>
                </a-button>
              </a-tooltip>
              <span class="mic-label">{{ isListening ? 'Listening...' : 'Tap Mic' }}</span>
            </div>
          </div>
          
          <div v-if="errorMessage" class="error-text">
            <ExclamationCircleOutlined /> {{ errorMessage }}
          </div>
        </a-form-item>

        <a-divider />

        <a-form-item class="actions-row">
          <div class="button-group">
            <a-button size="large" class="clear-btn" @click="clearForm">Clear</a-button>
            <a-button type="primary" size="large" class="publish-btn" @click="publishTopic">
              Publish Topic
            </a-button>
          </div>
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';
import { message } from 'ant-design-vue';
import { 
  AudioFilled, 
  ExclamationCircleOutlined 
} from '@ant-design/icons-vue';

const formState = reactive({
  title: '',
  description: '',
});

const currentLang = ref('en-US');
const isListening = ref(false);
const errorMessage = ref('');
let recognition = null;

onMounted(() => {
  if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    recognition = new SpeechRecognition();
    recognition.continuous = true;
    recognition.interimResults = true;
    recognition.lang = currentLang.value;

    recognition.onstart = () => {
      isListening.value = true;
      errorMessage.value = '';
    };

    recognition.onend = () => {
      isListening.value = false;
    };

    recognition.onerror = (event) => {
      isListening.value = false;
      errorMessage.value = `Error: ${event.error}`;
      if (event.error === 'not-allowed') {
        errorMessage.value = 'Microphone access denied.';
      }
    };

    recognition.onresult = (event) => {
      let finalTranscript = '';
      for (let i = event.resultIndex; i < event.results.length; ++i) {
        if (event.results[i].isFinal) {
          finalTranscript += event.results[i][0].transcript;
        }
      }
      if (finalTranscript) {
         const prefix = formState.description && !formState.description.endsWith(' ') ? ' ' : '';
         formState.description += prefix + finalTranscript;
      }
    };
  } else {
    errorMessage.value = 'Speech Recognition API not supported in this browser.';
  }
});

const handleLanguageChange = () => {
  if (recognition) {
    recognition.lang = currentLang.value;
    if (isListening.value) {
      recognition.stop();
      message.loading('Switching language...', 0.5).then(() => {
         message.info(`Language set to ${currentLang.value}. Press Mic to start.`);
      });
    }
  }
};

const toggleListening = () => {
  if (!recognition) {
    message.error('Speech Recognition not supported.');
    return;
  }
  recognition.lang = currentLang.value;
  if (isListening.value) {
    recognition.stop();
  } else {
    recognition.start();
  }
};

const publishTopic = () => {
  if (!formState.title || !formState.description) {
    message.warning('Please fill in both fields.');
    return;
  }
  console.log('Publishing Topic:', JSON.parse(JSON.stringify(formState)));
  message.success('Topic Published Successfully!');
};

const clearForm = () => {
  formState.title = '';
  formState.description = '';
  message.info('Form cleared.');
};
</script>

<style scoped>
.topic-publisher-container {
  width: 100%;
  max-width: 650px;
  animation: fadeIn 0.6s ease-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.publisher-card {
  border-radius: 24px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  overflow: hidden;
}

.card-header {
  text-align: center;
  padding: 10px 0;
  border-bottom: 2px solid #f0f0f0;
  margin-bottom: 20px;
}

.header-title {
  display: block;
  font-size: 24px;
  font-weight: 700;
  color: #1a1a1a;
  margin-bottom: 4px;
}

.header-subtitle {
  display: block;
  font-size: 14px;
  color: #8c8c8c;
}

.publisher-form :deep(.ant-form-item-label > label) {
  font-size: 15px;
  font-weight: 600;
  color: #333;
}

.custom-input, .custom-select {
  border-radius: 12px;
}

.description-container {
  position: relative;
  border: 1px solid #d9d9d9;
  border-radius: 12px;
  padding: 4px;
  background: white;
  transition: all 0.3s;
}

.description-container:focus-within {
  border-color: #4096ff;
  box-shadow: 0 0 0 2px rgba(5, 145, 255, 0.1);
}

.custom-textarea {
  border: none !important;
  box-shadow: none !important;
  resize: none;
  border-radius: 8px;
  padding-right: 60px; /* Space for mic button */
}

.mic-controls {
  position: absolute;
  top: 12px;
  right: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  z-index: 10;
}

.mic-fab {
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border: none;
  background: #f5f5f5;
  color: #595959;
}

.mic-fab:hover {
  transform: scale(1.05);
  background: #e6e6e6;
  color: #1890ff;
}

.mic-fab.is-recording {
  background: #ff4d4f;
  color: white;
  box-shadow: 0 0 0 4px rgba(255, 77, 79, 0.2);
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0% { box-shadow: 0 0 0 0 rgba(255, 77, 79, 0.4); }
  70% { box-shadow: 0 0 0 10px rgba(255, 77, 79, 0); }
  100% { box-shadow: 0 0 0 0 rgba(255, 77, 79, 0); }
}

.mic-label {
  font-size: 10px;
  font-weight: 600;
  color: #8c8c8c;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.waveform-icon {
  display: flex;
  align-items: center;
  gap: 3px;
  height: 16px;
}

.waveform-icon span {
  display: block;
  width: 3px;
  background: white;
  border-radius: 2px;
  animation: wave 1s infinite ease-in-out;
}

.waveform-icon span:nth-child(1) { height: 10px; animation-delay: 0s; }
.waveform-icon span:nth-child(2) { height: 16px; animation-delay: 0.1s; }
.waveform-icon span:nth-child(3) { height: 10px; animation-delay: 0.2s; }

@keyframes wave {
  0%, 100% { transform: scaleY(1); }
  50% { transform: scaleY(0.6); }
}

.error-text {
  margin-top: 8px;
  color: #ff4d4f;
  font-size: 13px;
  display: flex;
  align-items: center;
  gap: 4px;
}

.actions-row {
  margin-bottom: 0;
}

.button-group {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.publish-btn {
  padding: 0 32px;
  border-radius: 8px;
  font-weight: 600;
  background: linear-gradient(90deg, #1890ff 0%, #096dd9 100%);
  border: none;
  box-shadow: 0 4px 10px rgba(24, 144, 255, 0.3);
}

.publish-btn:hover {
  background: linear-gradient(90deg, #40a9ff 0%, #1890ff 100%);
  transform: translateY(-1px);
}

.clear-btn {
  border-radius: 8px;
}
</style>
