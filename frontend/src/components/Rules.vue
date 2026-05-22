<template>
  <v-container class="text-center fill-height bg-deep-dark" fluid>
    <v-row justify="center" align="center" class="w-100 py-8">
      <v-col cols="12" sm="11" md="10" lg="8">
        
        <div class="d-flex justify-space-between align-center mb-8">
          <v-btn icon="mdi-arrow-left" variant="text" color="grey" @click="router.push('/')"></v-btn>
          <h1 class="text-h3 font-weight-bold text-white title-glow text-uppercase">Spielregeln</h1>
          <div style="width: 48px;"></div> </div>

        <div class="lobby-wrapper pa-6 pa-md-10 text-left">
          
          <section class="mb-10">
            <h2 class="text-h4 text-cyan accent-glow mb-4">
              <v-icon color="#00e5ff" class="mr-2 pb-1">mdi-target</v-icon> Das Ziel: Die magische 5
            </h2>
            <p class="text-body-1 text-grey-lighten-1 mb-4" style="line-height: 1.8;">
              Bei <strong>Shift & Connect</strong> gewinnt der Spieler, der als Erstes 
              <strong>5 eigene Steine</strong> in einer ununterbrochenen Linie verbindet.
            </p>
            <div class="d-flex flex-wrap gap-4">
              <v-chip color="#00e5ff" variant="outlined" class="mr-2 mb-2 cyber-chip">
                <v-icon start>mdi-arrow-left-right</v-icon> Waagerecht
              </v-chip>
              <v-chip color="#ff9800" variant="outlined" class="mr-2 mb-2 cyber-chip">
                <v-icon start>mdi-arrow-up-down</v-icon> Senkrecht
              </v-chip>
              <v-chip color="#00e5ff" variant="outlined" class="mb-2 cyber-chip">
                <v-icon start>mdi-arrow-top-right-bottom-left</v-icon> Diagonal
              </v-chip>
            </div>
          </section>

          <v-divider class="border-opacity-25 mb-10" color="white"></v-divider>

          <section class="mb-10">
            <h2 class="text-h4 text-orange accent-glow mb-6">
              <v-icon color="#ff9800" class="mr-2 pb-1">mdi-gamepad-variant</v-icon> Dein Zug
            </h2>
            <p class="text-body-1 text-grey-lighten-1 mb-6" style="line-height: 1.8;">
              Gespielt wird abwechselnd. Wenn du am Zug bist, hast du genau <strong>zwei Möglichkeiten</strong>. 
              Du musst dich für eine davon entscheiden:
            </p>
            
            <v-row>
              <v-col cols="12" md="6">
                <v-card class="cyber-server-card pa-5 h-100" variant="outlined">
                  <div class="d-flex align-center mb-3">
                    <v-icon color="#00e5ff" size="large" class="mr-3">mdi-circle-slice-8</v-icon>
                    <h3 class="text-h6 text-white mb-0">1. Stein setzen</h3>
                  </div>
                  <p class="text-grey-lighten-1 text-body-2">
                    Lass deinen Spielstein in eine der Spalten fallen. Er rutscht so weit nach unten, bis er auf den Boden oder einen anderen Stein trifft.
                  </p>
                </v-card>
              </v-col>
              <v-col cols="12" md="6">
                <v-card class="cyber-server-card pa-5 h-100" variant="outlined">
                  <div class="d-flex align-center mb-3">
                    <v-icon color="#ff9800" size="large" class="mr-3">mdi-swap-horizontal</v-icon>
                    <h3 class="text-h6 text-white mb-0">2. Shiften</h3>
                  </div>
                  <p class="text-grey-lighten-1 text-body-2">
                    Anstatt einen Stein zu werfen, verschiebst du eine komplette Reihe oder Spalte um ein Feld, um das Board strategisch zu verändern.
                  </p>
                </v-card>
              </v-col>
            </v-row>
          </section>

          <v-divider class="border-opacity-25 mb-10" color="white"></v-divider>

          <section>
            <h2 class="text-h4 text-white accent-glow mb-6">
              <v-icon color="white" class="mr-2 pb-1">mdi-animation</v-icon> Wie funktioniert der Shift?
            </h2>

            <v-row>
              <v-col cols="12" md="6">
                <v-card class="cyber-server-card pa-5" variant="outlined">
                  <h3 class="text-h6 text-cyan mb-2">Der Horizontal-Shift</h3>
                  <p class="text-grey-lighten-1 text-body-2 mb-4" style="min-height: 60px;">
                    Schiebe eine gesamte Reihe nach links oder rechts. Steine, die am Rand hinausgeschoben werden, tauchen auf der anderen Seite der Reihe wieder auf.
                  </p>
                  
                  <div class="gif-wrapper rounded">
                    <img src="/src/assets/shift-horizontal.gif" alt="Horizontal Shift" class="rule-gif" @error="handleImageError" />
                    <div class="gif-placeholder bg-black d-flex align-center justify-center flex-column">
                      <v-icon color="#00e5ff" size="x-large" class="mb-2">mdi-arrow-left-right-bold</v-icon>
                      <span class="text-caption text-grey">@/assets/shift-horizontal.gif</span>
                    </div>
                  </div>
                </v-card>
              </v-col>

              <v-col cols="12" md="6">
                <v-card class="cyber-server-card pa-5" variant="outlined">
                  <h3 class="text-h6 text-orange mb-2">Der Vertikal-Shift</h3>
                  <p class="text-grey-lighten-1 text-body-2 mb-4" style="min-height: 60px;">
                    Bewege eine Spalte nach oben oder unten. Perfekt, um gegnerische 4er-Ketten aufzubrechen oder eigene Steine in Position fallen zu lassen.
                  </p>

                  <div class="gif-wrapper rounded">
                    <img src="/src/assets/shift-vertical.gif" alt="Vertical Shift" class="rule-gif" @error="handleImageError" />
                    <div class="gif-placeholder bg-black d-flex align-center justify-center flex-column">
                      <v-icon color="#ff9800" size="x-large" class="mb-2">mdi-arrow-up-down-bold</v-icon>
                      <span class="text-caption text-grey">@/assets/shift-vertical.gif</span>
                    </div>
                  </div>
                </v-card>
              </v-col>
            </v-row>
          </section>

        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup lang="ts">
import { useRouter } from 'vue-router';

const router = useRouter();

// Blendet den Placeholder ein, wenn die GIF-Datei noch nicht im assets-Ordner liegt.
const handleImageError = (e: Event) => {
  const target = e.target as HTMLElement;
  target.style.display = 'none'; 
};
</script>

<style scoped>
/* Globale Cyberpunk Styles (wie in Lobby/GameBoard) */
.bg-deep-dark {
  background-color: #121418;
  background-image: radial-gradient(circle at 50% 0%, #1f2532 0%, #121418 70%);
}

.title-glow {
  text-shadow: 0 0 20px rgba(255, 255, 255, 0.2);
  letter-spacing: 4px;
}

.accent-glow {
  text-shadow: 0 0 10px currentColor;
}

.text-cyan {
  color: #00e5ff !important;
}

.text-orange {
  color: #ff9800 !important;
}

.lobby-wrapper {
  background: linear-gradient(145deg, #252a35, #181a22);
  border-radius: 16px;
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.6), inset 0 1px 2px rgba(255, 255, 255, 0.1), inset 0 -1px 2px rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.05);
  overflow: hidden;
}

.cyber-server-card {
  background: rgba(0, 0, 0, 0.3) !important;
  border-color: rgba(255, 255, 255, 0.1) !important;
  transition: all 0.2s ease;
}

.cyber-server-card:hover {
  background: rgba(0, 0, 0, 0.5) !important;
  border-color: rgba(0, 229, 255, 0.3) !important;
}

.cyber-chip {
  background: rgba(0, 0, 0, 0.2);
}

/* GIF Wrapper / Platzhalter Styles */
.gif-wrapper {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  border: 1px solid rgba(255, 255, 255, 0.1);
  overflow: hidden;
}

.rule-gif {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.gif-placeholder {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1; /* Liegt hinter dem Image, wird sichtbar wenn Image ausfällt (display: none) */
}
</style>