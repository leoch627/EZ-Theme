<template>
  <div class="v2board-captcha">
    <div class="captcha-input-group">
      <div class="captcha-image-container">
        <img
          v-if="captchaImage"
          :src="captchaImage"
          alt="验证码"
          class="captcha-image"
          @click="refreshCaptcha"
        />
        <div v-else class="captcha-loading">
          <div class="loading-spinner"></div>
        </div>
        <button
          type="button"
          class="refresh-btn"
          @click="refreshCaptcha"
          :disabled="loading"
        >
          <IconRefresh :class="{ spinning: loading }" />
        </button>
      </div>
      <div class="captcha-input-container">
        <input
          v-model="captchaValue"
          type="text"
          :placeholder="$t('auth.captchaPlaceholder')"
          class="captcha-input"
          autocomplete="off"
          maxlength="4"
          @input="handleInput"
        />
      </div>
    </div>
    <p class="captcha-hint">{{ $t('auth.captchaHint') }}</p>
  </div>
</template>

<script>
import { ref, onMounted, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import IconRefresh from '@/components/icons/IconRefresh.vue'
import { useToast } from '@/composables/useToast'

export default {
  name: 'V2BoardCaptcha',
  components: {
    IconRefresh
  },
  props: {
    modelValue: {
      type: String,
      default: ''
    },
    captchaKey: {
      type: String,
      default: ''
    }
  },
  emits: ['update:modelValue', 'update:captchaKey', 'refresh'],
  setup(props, { emit }) {
    const { t } = useI18n()
    const { showToast } = useToast()
    
    const captchaImage = ref('')
    const captchaValue = ref(props.modelValue)
    const currentCaptchaKey = ref(props.captchaKey)
    const loading = ref(false)

    const handleInput = (event) => {
      captchaValue.value = event.target.value
      emit('update:modelValue', captchaValue.value)
    }

    const refreshCaptcha = async () => {
      if (loading.value) return
      
      loading.value = true
      
      try {
        const baseUrl = window.EZ_CONFIG?.API_CONFIG?.baseUrl || ''
        const response = await fetch(`${baseUrl}/api/v1/passport/captcha/generate`)
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`)
        }
        
        const data = await response.json()
        
        if (data.data && data.data.img && data.data.key) {
          captchaImage.value = data.data.img
          currentCaptchaKey.value = data.data.key
          captchaValue.value = ''
          
          emit('update:modelValue', '')
          emit('update:captchaKey', data.data.key)
          emit('refresh', data.data)
        } else {
          throw new Error('Invalid captcha response format')
        }
      } catch (error) {
        console.error('Failed to load captcha:', error)
        showToast(t('auth.captchaLoadFailed'), 'error')
      } finally {
        loading.value = false
      }
    }

    watch(() => props.modelValue, (newValue) => {
      captchaValue.value = newValue
    })

    watch(() => props.captchaKey, (newValue) => {
      currentCaptchaKey.value = newValue
    })

    onMounted(() => {
      refreshCaptcha()
    })

    return {
      captchaImage,
      captchaValue,
      currentCaptchaKey,
      loading,
      handleInput,
      refreshCaptcha
    }
  }
}
</script>

<style scoped>
.v2board-captcha {
  margin: 10px 0;
}

.captcha-input-group {
  display: flex;
  gap: 12px;
  align-items: stretch;
}

.captcha-image-container {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 120px;
  height: 40px;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  background: var(--card-background);
  overflow: hidden;
}

.captcha-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  cursor: pointer;
  user-select: none;
}

.captcha-loading {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.loading-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid var(--border-color);
  border-top: 2px solid var(--primary-color);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.refresh-btn {
  position: absolute;
  top: 2px;
  right: 2px;
  width: 24px;
  height: 24px;
  padding: 0;
  border: none;
  background: rgba(0, 0, 0, 0.6);
  color: white;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s;
}

.refresh-btn:hover {
  background: rgba(0, 0, 0, 0.8);
}

.refresh-btn:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

.refresh-btn .spinning {
  animation: spin 1s linear infinite;
}

.captcha-input-container {
  flex: 1;
}

.captcha-input {
  width: 100%;
  height: 40px;
  padding: 0 12px;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  background: var(--input-background);
  color: var(--text-color);
  font-size: 14px;
  transition: border-color 0.2s;
}

.captcha-input:focus {
  outline: none;
  border-color: var(--primary-color);
}

.captcha-hint {
  margin: 8px 0 0 0;
  font-size: 12px;
  color: var(--text-muted);
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@media (max-width: 768px) {
  .captcha-input-group {
    flex-direction: column;
    gap: 8px;
  }
  
  .captcha-image-container {
    width: 100%;
    height: 50px;
  }
}
</style>
