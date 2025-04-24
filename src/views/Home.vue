<template>
  <v-app>
    <v-container fluid >
      <v-row no-gutters class="fill-height align-center justify-center">
        <v-col cols="12" class="pa-4">
          <v-card
            class="mx-auto rounded-xl overflow-hidden"
            max-width="1280"
            elevation="12"
            :color="!scanComplete ? 'transparent' : undefined"
          >
            <v-row no-gutters>
              <v-col cols="12" md="5" class="pa-8 bg-primary-darken-1 position-relative overflow-hidden">
                <div class="position-relative" style="z-index: 2">
                  <div class="d-flex align-center mb-6">
                    <v-icon
                      icon="mdi-shield-search"
                      size="x-large"
                      color="light-blue-accent-3"
                      class="mr-3"
                    ></v-icon>
                    <h1 class="text-h3 font-weight-bold text-white">SecureScan</h1>
                  </div>

                  <p class="text-h6 text-blue-lighten-4 mb-8">
                    Complete security analysis platform to detect and neutralize digital threats.
                  </p>

                  <v-list bg-color="transparent" class="pa-0">
                    <v-list-item
                      v-for="(feature, i) in features"
                      :key="i"
                      class="px-0 py-2"
                    >
                      <template v-slot:prepend>
                        <v-avatar color="light-blue-accent-3" size="36" class="mr-3">
                          <v-icon :icon="feature.icon" color="white"></v-icon>
                        </v-avatar>
                      </template>
                      <v-list-item-title class="text-blue-lighten-4">
                        {{ feature.text }}
                      </v-list-item-title>
                    </v-list-item>
                  </v-list>
                </div>

                <div class="position-absolute circle-bg-1"></div>
                <div class="position-absolute circle-bg-2"></div>
              </v-col>

              <v-col cols="12" md="7" class="pa-8 bg-dark">
                <div v-if="scanComplete">
                  <div class="d-flex flex-column align-center py-4">
                    <v-avatar :color="threatColors[threatLevel].color" size="96" class="mb-6">
                      <v-icon :icon="threatColors[threatLevel].icon" size="x-large" color="white"></v-icon>
                    </v-avatar>

                    <h2 class="text-h4 font-weight-bold text-white mb-6">
                      {{ threatColors[threatLevel].text }}
                    </h2>

                    <v-card class="w-100 mb-6 rounded-lg" color="primary-darken-1" elevation="0">
                      <v-card-text>
                        <h3 class="text-h6 mb-4 text-white">Scan Results</h3>
                        <v-list bg-color="transparent" class="pa-0">
                          <v-list-item>
                            <v-list-item-title class="d-flex justify-space-between">
                              <span class="text-blue-lighten-3">Scan Type:</span>
                              <span class="text-white font-weight-medium">{{ tab.toUpperCase() }}</span>
                            </v-list-item-title>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-title class="d-flex justify-space-between">
                              <span class="text-blue-lighten-3">Target:</span>
                              <span class="text-white font-weight-medium text-truncate" style="max-width: 200px">
                                {{ scanTarget }}
                              </span>
                            </v-list-item-title>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-title class="d-flex justify-space-between">
                              <span class="text-blue-lighten-3">Threats Found:</span>
                              <span class="text-white font-weight-medium">
                                {{ threatLevel === 0 ? '0' : threatLevel === 1 ? '2' : '7' }}
                              </span>
                            </v-list-item-title>
                          </v-list-item>
                        </v-list>
                      </v-card-text>
                    </v-card>

                    <v-btn color="primary" variant="elevated" size="large" block @click="resetScan" class="text-none rounded-lg py-3">
                      New Scan
                    </v-btn>
                  </div>
                </div>

                <div v-else>
                  <v-tabs v-model="tab" color="primary" align-tabs="center" class="mb-6">
                    <v-tab value="file" class="text-none px-6">File</v-tab>
                    <v-tab value="url" class="text-none px-6">URL</v-tab>
                    <v-tab value="ip" class="text-none px-6">IP</v-tab>
                  </v-tabs>

                  <v-window v-model="tab" class="mb-6">
                    <v-window-item value="file">
                      <v-sheet color="primary-darken-2" class="pa-8 rounded-lg border border-dashed text-center" border-color="primary-lighten-1" height="240">
                        <div class="d-flex flex-column align-center justify-center h-100">
                          <v-icon icon="mdi-upload" color="primary-lighten-2" size="56" class="mb-4"></v-icon>
                          <div class="text-body-1 text-blue-lighten-3 mb-2">Drag and drop a file here or</div>
                          <v-btn color="primary" variant="elevated" @click="triggerFileInput" class="text-none mb-4">Browse Files</v-btn>
                          <div class="text-caption text-blue-lighten-4">Max file size: 100MB</div>
                          <input ref="fileInput" type="file" @change="onFileSelected" style="display: none" />
                        </div>
                      </v-sheet>

                      <div v-if="file" class="d-flex align-center mt-4 pa-2 bg-primary-darken-1 rounded">
                        <v-icon icon="mdi-file-outline" class="mr-2" color="light-blue-lighten-3"></v-icon>
                        <span class="text-truncate">{{ file.name }}</span>
                        <v-spacer></v-spacer>
                        <v-btn icon="mdi-close" variant="text" density="comfortable" color="primary-lighten-1" @click="file = null"></v-btn>
                      </div>
                    </v-window-item>

                    <v-window-item value="url">
                      <v-sheet color="transparent">
                        <div class="text-body-2 font-weight-medium text-blue-lighten-3 mb-2">Enter a URL to scan</div>
                        <v-text-field
                          v-model="url"
                          placeholder="https://example.com"
                          variant="outlined"
                          density="comfortable"
                          bg-color="primary-darken-2"
                          color="primary"
                          hide-details
                          class="rounded-lg mb-2"
                          prepend-inner-icon="mdi-link"
                        ></v-text-field>
                        <div class="text-caption text-blue-lighten-4">
                          We will check this URL for malicious content and security risks
                        </div>
                      </v-sheet>
                    </v-window-item>

                    <v-window-item value="ip">
                      <v-sheet color="transparent">
                        <div class="text-body-2 font-weight-medium text-blue-lighten-3 mb-2">Enter an IP address to analyze</div>
                        <v-text-field
                          v-model="ip"
                          placeholder="192.168.1.1"
                          variant="outlined"
                          density="comfortable"
                          bg-color="primary-darken-2"
                          color="primary"
                          hide-details
                          class="rounded-lg mb-2"
                          prepend-inner-icon="mdi-lan"
                        ></v-text-field>
                        <div class="text-caption text-blue-lighten-4">
                          We will check this IP for reputation, geolocation, and potential threats
                        </div>
                      </v-sheet>
                    </v-window-item>
                  </v-window>

                  <v-btn color="primary" variant="elevated" size="large" block :loading="isScanning" :disabled="!canScan" @click="startScan" class="text-none rounded-lg py-3">
                    <span v-if="!isScanning">Start Scanning</span>
                    <template v-else>Scanning...</template>
                  </v-btn>

                  <div class="text-caption text-center text-blue-lighten-4 mt-4">
                    By submitting, you agree to the terms of use. Do not upload personal or sensitive data.
                  </div>
                </div>
              </v-col>
            </v-row>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script setup>
import { ref, computed } from 'vue';

const tab = ref('file');
const file = ref(null);
const url = ref('');
const ip = ref('');
const isScanning = ref(false);
const scanComplete = ref(false);
const threatLevel = ref(0);
const fileInput = ref(null);

const features = [
  { icon: 'mdi-shield-search', text: 'Advanced threat detection algorithms' },
  { icon: 'mdi-clock-fast', text: 'Real-time scanning capabilities' },
  { icon: 'mdi-earth', text: 'Global threat intelligence network' },
];

const threatColors = [
  { color: 'success', icon: 'mdi-check-circle', text: 'No threats detected' },
  { color: 'warning', icon: 'mdi-alert-circle', text: 'Potential risks detected' },
  { color: 'error', icon: 'mdi-alert', text: 'Serious threats detected' },
];

const canScan = computed(() => {
  if (tab.value === 'file') return !!file.value;
  if (tab.value === 'url') return !!url.value;
  if (tab.value === 'ip') return !!ip.value;
  return false;
});

const scanTarget = computed(() => {
  if (tab.value === 'file') return file.value ? file.value.name : 'N/A';
  if (tab.value === 'url') return url.value || 'N/A';
  if (tab.value === 'ip') return ip.value || 'N/A';
  return 'N/A';
});

const triggerFileInput = () => fileInput.value.click();
const onFileSelected = (event) => file.value = event.target.files[0] || null;
const startScan = () => {
  isScanning.value = true;
  setTimeout(() => {
    isScanning.value = false;
    scanComplete.value = true;
    threatLevel.value = Math.floor(Math.random() * 3);
  }, 2000);
};
const resetScan = () => {
  scanComplete.value = false;
  threatLevel.value = 0;
  if (tab.value === 'file') file.value = null;
  else if (tab.value === 'url') url.value = '';
  else if (tab.value === 'ip') ip.value = '';
};
</script>

<style>
.bg-gradient {
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 50%, #334155 100%);
}
.bg-dark {
  background-color: #101828 !important;
}
.circle-bg-1 {
  width: 300px;
  height: 300px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(33, 150, 243, 0.1) 0%, rgba(33, 150, 243, 0) 70%);
  top: -100px;
  right: -50px;
  z-index: 1;
}
.circle-bg-2 {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(3, 169, 244, 0.1) 0%, rgba(3, 169, 244, 0) 70%);
  bottom: -50px;
  left: -50px;
  z-index: 1;
}
</style>