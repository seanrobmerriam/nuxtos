<script setup lang="ts">
type DesktopApp = {
  id: string
  label: string
  icon: string
}

const apps: DesktopApp[] = [
  {
    id: 'home',
    label: 'Home',
    icon: 'i-lucide-house'
  },
  {
    id: 'files',
    label: 'Files',
    icon: 'i-lucide-folder'
  },
  {
    id: 'terminal',
    label: 'Terminal',
    icon: 'i-lucide-terminal'
  },
  {
    id: 'messages',
    label: 'Messages',
    icon: 'i-lucide-message-circle'
  },
  {
    id: 'settings',
    label: 'Settings',
    icon: 'i-lucide-settings'
  }
]

const activeAppId = ref<string | null>('home')
const currentTime = ref('')

const activeApp = computed(() => {
  return apps.find(app => app.id === activeAppId.value) ?? null
})

const desktopApps = computed(() => {
  return apps.filter(app => app.id !== 'home')
})

const windowElement = ref<HTMLElement | null>(null)

const windowPosition = reactive({
  x: 120,
  y: 90
})

let clockTimer: ReturnType<typeof setInterval> | undefined

let isDragging = false
let dragOffsetX = 0
let dragOffsetY = 0

function updateClock() {
  currentTime.value = new Intl.DateTimeFormat('en-US', {
    weekday: 'short',
    month: 'short',
    day: 'numeric',
    hour: 'numeric',
    minute: '2-digit'
  }).format(new Date())
}

function openApp(id: string) {
  activeAppId.value = id
}

function closeApp() {
  activeAppId.value = null
}

function centerWindow() {
  if (!import.meta.client) {
    return
  }

  const width = Math.min(760, window.innerWidth - 48)
  const height = Math.min(520, window.innerHeight - 160)

  windowPosition.x = Math.max(24, (window.innerWidth - width) / 2)
  windowPosition.y = Math.max(48, (window.innerHeight - height) / 2)
}

function startDragging(event: PointerEvent) {
  if (window.innerWidth < 640) {
    return
  }

  isDragging = true
  dragOffsetX = event.clientX - windowPosition.x
  dragOffsetY = event.clientY - windowPosition.y

  window.addEventListener('pointermove', dragWindow)
  window.addEventListener('pointerup', stopDragging)
}

function dragWindow(event: PointerEvent) {
  if (!isDragging || !windowElement.value) {
    return
  }

  const windowWidth = windowElement.value.offsetWidth
  const windowHeight = windowElement.value.offsetHeight

  const maximumX = Math.max(12, window.innerWidth - windowWidth - 12)
  const maximumY = Math.max(48, window.innerHeight - windowHeight - 96)

  windowPosition.x = Math.min(
    Math.max(12, event.clientX - dragOffsetX),
    maximumX
  )

  windowPosition.y = Math.min(
    Math.max(48, event.clientY - dragOffsetY),
    maximumY
  )
}

function stopDragging() {
  isDragging = false

  window.removeEventListener('pointermove', dragWindow)
  window.removeEventListener('pointerup', stopDragging)
}

onMounted(() => {
  updateClock()
  centerWindow()

  clockTimer = setInterval(updateClock, 30_000)

  window.addEventListener('resize', centerWindow)
})

onBeforeUnmount(() => {
  if (clockTimer) {
    clearInterval(clockTimer)
  }

  window.removeEventListener('resize', centerWindow)
  window.removeEventListener('pointermove', dragWindow)
  window.removeEventListener('pointerup', stopDragging)
})
</script>

<template>
  <UApp>
    <main class="desktop">
      <!-- Wallpaper decoration -->
      <div class="wallpaper-glow wallpaper-glow-one" />
      <div class="wallpaper-glow wallpaper-glow-two" />
      <div class="wallpaper-grid" />

      <!-- Top system bar -->
      <header class="system-bar">
        <div class="system-bar-section">
          <UIcon
            name="i-lucide-monitor"
            class="size-4"
          />

          <span class="font-semibold">
            Nuxt Desktop
          </span>

          <button
            type="button"
            class="system-menu-item"
          >
            File
          </button>

          <button
            type="button"
            class="system-menu-item"
          >
            View
          </button>
        </div>

        <div class="system-bar-section">
          <UIcon
            name="i-lucide-wifi"
            class="size-4"
          />

          <UIcon
            name="i-lucide-battery-full"
            class="size-4"
          />

          <span class="hidden sm:inline">
            {{ currentTime }}
          </span>
        </div>
      </header>

      <!-- Desktop shortcuts -->
      <div class="desktop-shortcuts">
        <button
          v-for="app in desktopApps"
          :key="app.id"
          type="button"
          class="desktop-shortcut"
          @click="openApp(app.id)"
          @dblclick="openApp(app.id)"
        >
          <span class="desktop-shortcut-icon">
            <UIcon
              :name="app.icon"
              class="size-8"
            />
          </span>

          <span class="desktop-shortcut-label">
            {{ app.label }}
          </span>
        </button>
      </div>

      <!-- Application window -->
      <Transition name="window">
        <section
          v-if="activeApp"
          ref="windowElement"
          class="app-window"
          :style="{
            left: `${windowPosition.x}px`,
            top: `${windowPosition.y}px`
          }"
        >
          <!-- Window title bar -->
          <header
            class="window-titlebar"
            @pointerdown="startDragging"
          >
            <div class="window-title">
              <span class="window-app-icon">
                <UIcon
                  :name="activeApp.icon"
                  class="size-4"
                />
              </span>

              <span>{{ activeApp.label }}</span>
            </div>

            <div
              class="window-actions"
              @pointerdown.stop
            >
              <UButton
                icon="i-lucide-minus"
                color="neutral"
                variant="ghost"
                size="xs"
                square
                aria-label="Minimize application"
                @click="closeApp"
              />

              <UButton
                icon="i-lucide-maximize-2"
                color="neutral"
                variant="ghost"
                size="xs"
                square
                aria-label="Center application"
                @click="centerWindow"
              />

              <UButton
                icon="i-lucide-x"
                color="error"
                variant="ghost"
                size="xs"
                square
                aria-label="Close application"
                @click="closeApp"
              />
            </div>
          </header>

          <!-- Window content -->
          <div class="window-content">
            <!-- Home -->
            <template v-if="activeApp.id === 'home'">
              <div class="welcome-header">
                <div>
                  <UBadge
                    color="primary"
                    variant="subtle"
                    label="Nuxt UI Desktop"
                  />

                  <h1 class="mt-4 text-3xl font-semibold tracking-tight">
                    Welcome back
                  </h1>

                  <p class="mt-2 max-w-xl text-sm text-muted">
                    This application uses a desktop-style interface with
                    launchable applications and a persistent dock.
                  </p>
                </div>

                <div class="welcome-logo">
                  <UIcon
                    name="i-simple-icons-nuxtdotjs"
                    class="size-12"
                  />
                </div>
              </div>

              <div class="dashboard-grid">
                <button
                  v-for="app in desktopApps"
                  :key="app.id"
                  type="button"
                  class="dashboard-card"
                  @click="openApp(app.id)"
                >
                  <span class="dashboard-card-icon">
                    <UIcon
                      :name="app.icon"
                      class="size-6"
                    />
                  </span>

                  <span>
                    <strong>{{ app.label }}</strong>
                    <small>Open application</small>
                  </span>

                  <UIcon
                    name="i-lucide-chevron-right"
                    class="ml-auto size-4 text-dimmed"
                  />
                </button>
              </div>
            </template>

            <!-- Files -->
            <template v-else-if="activeApp.id === 'files'">
              <div class="content-toolbar">
                <div>
                  <h2 class="text-lg font-semibold">
                    Files
                  </h2>

                  <p class="text-sm text-muted">
                    Your recent directories and documents.
                  </p>
                </div>

                <UButton
                  label="New folder"
                  icon="i-lucide-folder-plus"
                  size="sm"
                />
              </div>

              <div class="file-grid">
                <button
                  v-for="folder in ['Projects', 'Documents', 'Images', 'Downloads']"
                  :key="folder"
                  type="button"
                  class="file-card"
                >
                  <UIcon
                    name="i-lucide-folder"
                    class="size-10 text-primary"
                  />

                  <span class="font-medium">
                    {{ folder }}
                  </span>

                  <span class="text-xs text-muted">
                    Folder
                  </span>
                </button>
              </div>
            </template>

            <!-- Terminal -->
            <template v-else-if="activeApp.id === 'terminal'">
              <div class="terminal">
                <div class="terminal-line">
                  <span class="terminal-prompt">sean@nuxt-desktop</span>
                  <span>:</span>
                  <span class="terminal-path">~/projects</span>
                  <span>$</span>
                  <span>pnpm dev</span>
                </div>

                <p>Nuxt development server started.</p>
                <p>Local: http://localhost:3000</p>
                <p class="text-emerald-400">
                  ✓ Vite client built successfully
                </p>

                <div class="terminal-line">
                  <span class="terminal-prompt">sean@nuxt-desktop</span>
                  <span>:</span>
                  <span class="terminal-path">~/projects</span>
                  <span>$</span>
                  <span class="terminal-cursor" />
                </div>
              </div>
            </template>

            <!-- Messages -->
            <template v-else-if="activeApp.id === 'messages'">
              <div class="empty-state">
                <span class="empty-state-icon">
                  <UIcon
                    name="i-lucide-message-circle"
                    class="size-10"
                  />
                </span>

                <h2 class="text-lg font-semibold">
                  No new messages
                </h2>

                <p class="max-w-sm text-center text-sm text-muted">
                  Your conversations will appear here when the messaging
                  service is connected.
                </p>

                <UButton
                  label="Start conversation"
                  icon="i-lucide-plus"
                />
              </div>
            </template>

            <!-- Settings -->
            <template v-else-if="activeApp.id === 'settings'">
              <div class="content-toolbar">
                <div>
                  <h2 class="text-lg font-semibold">
                    Settings
                  </h2>

                  <p class="text-sm text-muted">
                    Customize your desktop environment.
                  </p>
                </div>
              </div>

              <div class="settings-list">
                <button
                  v-for="setting in [
                    {
                      label: 'Appearance',
                      description: 'Wallpaper, colors and visual preferences',
                      icon: 'i-lucide-palette'
                    },
                    {
                      label: 'Applications',
                      description: 'Manage installed desktop applications',
                      icon: 'i-lucide-layout-grid'
                    },
                    {
                      label: 'Notifications',
                      description: 'Control application notifications',
                      icon: 'i-lucide-bell'
                    }
                  ]"
                  :key="setting.label"
                  type="button"
                  class="settings-row"
                >
                  <span class="settings-icon">
                    <UIcon
                      :name="setting.icon"
                      class="size-5"
                    />
                  </span>

                  <span>
                    <strong>{{ setting.label }}</strong>
                    <small>{{ setting.description }}</small>
                  </span>

                  <UIcon
                    name="i-lucide-chevron-right"
                    class="ml-auto size-4 text-dimmed"
                  />
                </button>
              </div>
            </template>
          </div>
        </section>
      </Transition>

      <!-- Bottom dock -->
      <nav
        class="dock"
        aria-label="Application dock"
      >
        <div
          v-for="app in apps"
          :key="app.id"
          class="dock-item-wrapper"
        >
          <UTooltip
            :text="app.label"
            :delay-duration="0"
          >
            <div class="relative">
              <UButton
                :icon="app.icon"
                color="neutral"
                variant="ghost"
                size="xl"
                square
                :aria-label="`Open ${app.label}`"
                class="dock-item"
                :class="{
                  'dock-item-active': activeAppId === app.id
                }"
                @click="openApp(app.id)"
              />

              <span
                v-if="activeAppId === app.id"
                class="dock-active-indicator"
              />
            </div>
          </UTooltip>
        </div>

        <div class="dock-separator" />

        <div class="dock-item-wrapper">
          <UTooltip
            text="Trash"
            :delay-duration="0"
          >
            <div>
              <UButton
                icon="i-lucide-trash-2"
                color="neutral"
                variant="ghost"
                size="xl"
                square
                aria-label="Open trash"
                class="dock-item"
              />
            </div>
          </UTooltip>
        </div>
      </nav>
    </main>
  </UApp>
</template>

<style scoped>
:global(html),
:global(body),
:global(#__nuxt) {
  width: 100%;
  height: 100%;
  margin: 0;
  overflow: hidden;
}

.desktop {
  position: relative;
  width: 100%;
  min-height: 100dvh;
  overflow: hidden;
  color: white;
  background:
    radial-gradient(circle at 15% 20%, rgb(56 189 248 / 30%), transparent 32%),
    radial-gradient(circle at 80% 30%, rgb(168 85 247 / 30%), transparent 32%),
    linear-gradient(145deg, #09111f 0%, #111b34 50%, #141126 100%);
}

.wallpaper-grid {
  position: absolute;
  inset: 0;
  opacity: 0.08;
  pointer-events: none;
  background-image:
    linear-gradient(rgb(255 255 255 / 20%) 1px, transparent 1px),
    linear-gradient(90deg, rgb(255 255 255 / 20%) 1px, transparent 1px);
  background-size: 40px 40px;
  mask-image: linear-gradient(to bottom, black, transparent 85%);
}

.wallpaper-glow {
  position: absolute;
  width: 32rem;
  height: 32rem;
  border-radius: 9999px;
  filter: blur(100px);
  opacity: 0.2;
  pointer-events: none;
}

.wallpaper-glow-one {
  top: -12rem;
  left: -8rem;
  background: #0ea5e9;
}

.wallpaper-glow-two {
  right: -10rem;
  bottom: -14rem;
  background: #8b5cf6;
}

.system-bar {
  position: absolute;
  z-index: 50;
  top: 0;
  left: 0;
  display: flex;
  width: 100%;
  height: 2.25rem;
  align-items: center;
  justify-content: space-between;
  padding: 0 0.85rem;
  border-bottom: 1px solid rgb(255 255 255 / 10%);
  background: rgb(10 15 28 / 55%);
  box-shadow: 0 8px 30px rgb(0 0 0 / 10%);
  backdrop-filter: blur(24px);
  font-size: 0.75rem;
  user-select: none;
}

.system-bar-section {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.system-menu-item {
  color: rgb(255 255 255 / 75%);
  transition:
    color 150ms ease,
    background-color 150ms ease;
}

.system-menu-item:hover {
  color: white;
}

.desktop-shortcuts {
  position: absolute;
  z-index: 5;
  top: 3.5rem;
  left: 1rem;
  display: grid;
  gap: 0.8rem;
}

.desktop-shortcut {
  display: flex;
  width: 5rem;
  flex-direction: column;
  align-items: center;
  gap: 0.35rem;
  border-radius: 0.75rem;
  padding: 0.45rem;
  outline: none;
  transition:
    background-color 150ms ease,
    transform 150ms ease;
}

.desktop-shortcut:hover,
.desktop-shortcut:focus-visible {
  background: rgb(255 255 255 / 10%);
  transform: translateY(-1px);
}

.desktop-shortcut-icon {
  display: grid;
  width: 3.5rem;
  height: 3.5rem;
  place-items: center;
  border: 1px solid rgb(255 255 255 / 15%);
  border-radius: 1rem;
  background:
    linear-gradient(
      145deg,
      rgb(255 255 255 / 25%),
      rgb(255 255 255 / 8%)
    );
  box-shadow:
    inset 0 1px 0 rgb(255 255 255 / 20%),
    0 12px 30px rgb(0 0 0 / 25%);
  backdrop-filter: blur(16px);
}

.desktop-shortcut-label {
  max-width: 100%;
  overflow: hidden;
  color: rgb(255 255 255 / 90%);
  font-size: 0.7rem;
  text-overflow: ellipsis;
  text-shadow: 0 1px 4px rgb(0 0 0 / 80%);
  white-space: nowrap;
}

.app-window {
  position: absolute;
  z-index: 20;
  display: flex;
  width: min(760px, calc(100vw - 48px));
  height: min(520px, calc(100dvh - 160px));
  min-height: 380px;
  flex-direction: column;
  overflow: hidden;
  border: 1px solid rgb(255 255 255 / 15%);
  border-radius: 1rem;
  background: rgb(10 15 28 / 82%);
  box-shadow:
    0 32px 80px rgb(0 0 0 / 45%),
    inset 0 1px 0 rgb(255 255 255 / 12%);
  backdrop-filter: blur(28px);
}

.window-titlebar {
  display: flex;
  min-height: 3rem;
  cursor: grab;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid rgb(255 255 255 / 10%);
  padding: 0 0.65rem 0 0.9rem;
  background: rgb(255 255 255 / 4%);
  user-select: none;
  touch-action: none;
}

.window-titlebar:active {
  cursor: grabbing;
}

.window-title {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  font-size: 0.8rem;
  font-weight: 600;
}

.window-app-icon {
  display: grid;
  width: 1.75rem;
  height: 1.75rem;
  place-items: center;
  border: 1px solid rgb(255 255 255 / 10%);
  border-radius: 0.5rem;
  background: rgb(255 255 255 / 8%);
}

.window-actions {
  display: flex;
  align-items: center;
  gap: 0.2rem;
}

.window-content {
  min-height: 0;
  flex: 1;
  overflow: auto;
  padding: 1.5rem;
}

.welcome-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 2rem;
}

.welcome-logo {
  display: grid;
  width: 6rem;
  height: 6rem;
  flex: none;
  place-items: center;
  border: 1px solid rgb(255 255 255 / 12%);
  border-radius: 1.5rem;
  color: #00dc82;
  background: rgb(255 255 255 / 6%);
  box-shadow: inset 0 1px 0 rgb(255 255 255 / 12%);
}

.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.75rem;
  margin-top: 2rem;
}

.dashboard-card,
.settings-row {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  border: 1px solid rgb(255 255 255 / 10%);
  border-radius: 0.85rem;
  padding: 0.85rem;
  text-align: left;
  background: rgb(255 255 255 / 5%);
  transition:
    border-color 150ms ease,
    background-color 150ms ease,
    transform 150ms ease;
}

.dashboard-card:hover,
.settings-row:hover {
  border-color: rgb(255 255 255 / 20%);
  background: rgb(255 255 255 / 9%);
  transform: translateY(-1px);
}

.dashboard-card-icon,
.settings-icon {
  display: grid;
  width: 2.75rem;
  height: 2.75rem;
  flex: none;
  place-items: center;
  border-radius: 0.75rem;
  color: rgb(255 255 255 / 90%);
  background: rgb(255 255 255 / 8%);
}

.dashboard-card strong,
.dashboard-card small,
.settings-row strong,
.settings-row small {
  display: block;
}

.dashboard-card strong,
.settings-row strong {
  font-size: 0.85rem;
}

.dashboard-card small,
.settings-row small {
  margin-top: 0.15rem;
  color: rgb(255 255 255 / 45%);
  font-size: 0.7rem;
}

.content-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.file-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 0.75rem;
  margin-top: 1.5rem;
}

.file-card {
  display: flex;
  min-height: 8.5rem;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.45rem;
  border: 1px solid rgb(255 255 255 / 10%);
  border-radius: 0.9rem;
  background: rgb(255 255 255 / 5%);
  transition:
    border-color 150ms ease,
    background-color 150ms ease,
    transform 150ms ease;
}

.file-card:hover {
  border-color: rgb(255 255 255 / 20%);
  background: rgb(255 255 255 / 9%);
  transform: translateY(-2px);
}

.terminal {
  min-height: 100%;
  border: 1px solid rgb(255 255 255 / 8%);
  border-radius: 0.8rem;
  padding: 1rem;
  color: rgb(226 232 240);
  background: rgb(0 0 0 / 45%);
  font-family:
    ui-monospace,
    SFMono-Regular,
    Menlo,
    Monaco,
    Consolas,
    monospace;
  font-size: 0.78rem;
  line-height: 1.75;
}

.terminal-line {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
}

.terminal-prompt {
  color: #34d399;
}

.terminal-path {
  color: #60a5fa;
}

.terminal-cursor {
  display: inline-block;
  width: 0.5rem;
  height: 1rem;
  margin-top: 0.25rem;
  background: white;
  animation: cursor-blink 1s steps(1) infinite;
}

.empty-state {
  display: flex;
  min-height: 100%;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.9rem;
}

.empty-state-icon {
  display: grid;
  width: 5rem;
  height: 5rem;
  place-items: center;
  border: 1px solid rgb(255 255 255 / 10%);
  border-radius: 1.25rem;
  color: rgb(255 255 255 / 80%);
  background: rgb(255 255 255 / 6%);
}

.settings-list {
  display: grid;
  gap: 0.65rem;
  margin-top: 1.5rem;
}

.dock {
  position: absolute;
  z-index: 50;
  bottom: max(1rem, env(safe-area-inset-bottom));
  left: 50%;
  display: flex;
  align-items: flex-end;
  gap: 0.35rem;
  max-width: calc(100vw - 1.5rem);
  padding: 0.45rem;
  border: 1px solid rgb(255 255 255 / 16%);
  border-radius: 1.25rem;
  background: rgb(15 23 42 / 55%);
  box-shadow:
    0 20px 50px rgb(0 0 0 / 35%),
    inset 0 1px 0 rgb(255 255 255 / 15%);
  backdrop-filter: blur(24px);
  transform: translateX(-50%);
}

.dock-item-wrapper {
  position: relative;
  flex: none;
}

.dock-item {
  width: 3rem;
  height: 3rem;
  border: 1px solid rgb(255 255 255 / 10%);
  border-radius: 0.9rem;
  color: rgb(255 255 255 / 85%);
  background:
    linear-gradient(
      145deg,
      rgb(255 255 255 / 15%),
      rgb(255 255 255 / 5%)
    );
  box-shadow:
    inset 0 1px 0 rgb(255 255 255 / 12%),
    0 8px 18px rgb(0 0 0 / 15%);
  transition:
    transform 180ms cubic-bezier(0.22, 1, 0.36, 1),
    background-color 150ms ease,
    border-color 150ms ease;
}

.dock-item-wrapper:hover .dock-item {
  border-color: rgb(255 255 255 / 24%);
  background: rgb(255 255 255 / 16%);
  transform: translateY(-9px) scale(1.15);
}

.dock-item-active {
  background: rgb(255 255 255 / 16%);
}

.dock-active-indicator {
  position: absolute;
  bottom: -0.25rem;
  left: 50%;
  width: 0.28rem;
  height: 0.28rem;
  border-radius: 9999px;
  background: white;
  box-shadow: 0 0 8px rgb(255 255 255 / 75%);
  transform: translateX(-50%);
}

.dock-separator {
  width: 1px;
  height: 2.5rem;
  margin: 0 0.15rem;
  background: rgb(255 255 255 / 18%);
}

.window-enter-active,
.window-leave-active {
  transition:
    opacity 180ms ease,
    transform 180ms cubic-bezier(0.22, 1, 0.36, 1);
}

.window-enter-from,
.window-leave-to {
  opacity: 0;
  transform: translateY(16px) scale(0.96);
}

@keyframes cursor-blink {
  0%,
  49% {
    opacity: 1;
  }

  50%,
  100% {
    opacity: 0;
  }
}

@media (max-width: 639px) {
  .desktop-shortcuts {
    display: none;
  }

  .app-window {
    inset: 3rem 0.75rem 6.25rem !important;
    width: auto !important;
    height: auto;
    min-height: 0;
  }

  .window-titlebar {
    cursor: default;
  }

  .window-content {
    padding: 1rem;
  }

  .welcome-logo {
    display: none;
  }

  .dashboard-grid {
    grid-template-columns: 1fr;
  }

  .file-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .dock {
    gap: 0.2rem;
    padding: 0.35rem;
  }

  .dock-item {
    width: 2.65rem;
    height: 2.65rem;
  }

  .dock-item-wrapper:hover .dock-item {
    transform: none;
  }
}
</style>