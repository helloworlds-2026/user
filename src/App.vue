<template>
  <!-- Config loading screen -->
  <div v-if="!appStore.config" class="fixed inset-0 theme-page flex items-center justify-center z-[9999]">
    <div class="flex flex-col items-center gap-4">
      <svg class="w-10 h-10 animate-spin text-gray-400 dark:text-gray-500" fill="none" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" />
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z" />
      </svg>
    </div>
  </div>

  <div v-else id="app" class="min-h-screen theme-page flex flex-col">
    <Navbar />
    <main class="flex-1 pb-14 lg:pb-0">
      <ErrorBoundary>
        <RouterView v-slot="{ Component }">
          <Transition name="page-fade" mode="out-in">
            <component :is="Component" />
          </Transition>
        </RouterView>
      </ErrorBoundary>
    </main>

    <!-- ✅ Footer 已移除 -->

    <Loading :loading="appStore.loading" />
    <Toast />
    <ConfirmDialog />
    <BackToTop />
    <MobileBottomNav />
  </div>
</template>

<script setup lang="ts">
import { useAppStore } from './stores/app'
import Navbar from './components/Navbar.vue'
import Loading from './components/Loading.vue'
import Toast from './components/Toast.vue'
import ConfirmDialog from './components/ConfirmDialog.vue'
import ErrorBoundary from './components/ErrorBoundary.vue'
import BackToTop from './components/BackToTop.vue'
import MobileBottomNav from './components/MobileBottomNav.vue'

// config 由 router.beforeEach 统一加载，无需在此重复调用
const appStore = useAppStore()
</script>

<style>
.page-fade-enter-active,
.page-fade-leave-active {
  transition: opacity 200ms ease;
}

.page-fade-enter-from,
.page-fade-leave-to {
  opacity: 0;
}
</style>
