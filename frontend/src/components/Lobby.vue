<template>
  <v-container class="text-center fill-height bg-deep-dark" fluid>
    <v-row justify="center" align="center" class="w-100">
      <v-col cols="12" sm="10" md="10" lg="8">
        <h1 class="text-h3 font-weight-bold text-white mb-8 title-glow">SYSTEM UPLINK</h1>

        <div class="lobby-wrapper">
          <v-row no-gutters>
            <v-col cols="12" md="5" class="border-right-md pa-6 d-flex flex-column">
              <h2 class="text-h5 text-cyan accent-glow mb-6">Neues Signal</h2>
              
              <v-btn-toggle v-model="gameMode" mandatory class="cyber-toggle mb-4" selected-class="cyber-toggle-active">
                <v-btn value="duo" class="flex-grow-1"><v-icon start>mdi-account-multiple</v-icon> Duo</v-btn>
                <v-btn value="solo" class="flex-grow-1"><v-icon start>mdi-robot</v-icon> Solo</v-btn>
              </v-btn-toggle>

              <v-switch
                v-model="isPrivate"
                color="#00e5ff"
                label="Privates Signal (Passwort)"
                hide-details
                class="mb-8 cyber-switch"
              ></v-switch>

              <v-spacer></v-spacer>

              <v-btn block size="x-large" variant="outlined" class="cyber-btn-primary" @click="createGame">
                Initialisieren <v-icon end>mdi-rocket-launch</v-icon>
              </v-btn>
              <router-link to="/regeln">Spielregeln ansehen</router-link>
              
            </v-col>

            <v-col cols="12" md="7" class="pa-6">
              <div class="d-flex justify-space-between align-center mb-6">
                <h2 class="text-h5 text-orange accent-glow">Aktive Signale</h2>
                <v-btn icon="mdi-refresh" variant="text" color="#ff9800" @click="fetchGames"></v-btn>
              </div>
              
              <div class="server-list">
                <div v-if="gamesList.length === 0" class="text-grey text-center mt-8">
                  Keine aktiven Signale gefunden.
                </div>

                <v-card
                  v-for="game in gamesList"
                  :key="game.room_code"
                  class="cyber-server-card mb-3 pa-3 d-flex justify-space-between align-center"
                  variant="outlined"
                >
                  <div class="d-flex align-center">
                    <v-icon :color="game.is_private ? 'error' : 'success'" class="mr-3">
                      {{ game.is_private ? 'mdi-lock' : 'mdi-lock-open-variant' }}
                    </v-icon>
                    <div class="text-left">
                      <div class="text-h6 font-weight-bold">Raum: {{ game.room_code }}</div>
                      <div class="text-caption text-grey-lighten-1">Modus: {{ game.game_mode.toUpperCase() }}</div>
                    </div>
                  </div>
                  
                  <v-btn
                    icon="mdi-login"
                    variant="tonal"
                    :color="game.is_private ? 'error' : 'success'"
                    @click="initiateJoin(game)"
                  ></v-btn>
                </v-card>
              </div>
            </v-col>
          </v-row>
        </div>
      </v-col>
    </v-row>

    <v-dialog v-model="showCreatedPasswordDialog" max-width="400" persistent>
      <v-card class="bg-deep-dark cyber-dialog">
        <v-card-title class="text-cyan text-center pt-6">Signal verschlüsselt!</v-card-title>
        <v-card-text class="text-center pb-6">
          <p class="mb-4">Teile dieses Passwort mit deinem Mitspieler:</p>
          <div class="text-h3 font-weight-bold text-orange tracking-widest">{{ createdPassword }}</div>
        </v-card-text>
        <v-card-actions class="justify-center pb-6">
          <v-btn color="#00e5ff" variant="outlined" @click="joinCreatedGame">Feld betreten</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="showJoinDialog" max-width="400">
      <v-card class="bg-deep-dark cyber-dialog">
        <v-card-title class="text-orange text-center pt-6">Sicherheitsfreigabe</v-card-title>
        <v-card-text>
          <p class="text-center mb-4">Raum {{ selectedRoom }} erfordert ein Passwort.</p>
          <v-text-field
            v-model="inputPassword"
            label="Passwort"
            variant="outlined"
            color="#ff9800"
            base-color="rgba(255, 255, 255, 0.3)"
            class="cyber-input text-center"
            :error-messages="joinError"
            @keyup.enter="submitJoin"
          ></v-text-field>
        </v-card-text>
        <v-card-actions class="justify-center pb-6">
          <v-btn color="grey" variant="text" @click="showJoinDialog = false">Abbrechen</v-btn>
          <v-btn color="#ff9800" variant="outlined" @click="submitJoin">Entschlüsseln</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </v-container>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'; // onUnmounted hinzugefügt
import { useRouter } from 'vue-router';

const router = useRouter();

// State: Erstellung
const gameMode = ref<string>('duo');
const isPrivate = ref<boolean>(false);
const createdRoomCode = ref<string>('');
const createdPassword = ref<string>('');
const showCreatedPasswordDialog = ref<boolean>(false);

// State: Server Liste
interface GameListItem {
  room_code: string;
  game_mode: string;
  is_private: boolean; // boolean statt bool in TS!
}
const gamesList = ref<GameListItem[]>([]);

// State: Join Flow
const showJoinDialog = ref<boolean>(false);
const selectedRoom = ref<string>('');
const inputPassword = ref<string>('');
const joinError = ref<string>('');

// --- LOBBY ECHTZEIT-VERBINDUNG ---
let lobbyWs: WebSocket | null = null;

onMounted(() => {
  fetchGames(); // Initiales Laden

  // Verbinde dich mit dem Lobby-Kanal
  lobbyWs = new WebSocket('ws://127.0.0.1:3000/ws/lobby');
  
  lobbyWs.onmessage = (event) => {
    // Wenn der Server "update" schickt (neues Spiel, Spiel gewonnen oder Inaktivität)
    if (event.data === 'update') {
      fetchGames(); // Liste lautlos im Hintergrund aktualisieren
    }
  };
});

onUnmounted(() => {
  // Wenn der Spieler die Lobby verlässt, Verbindung trennen
  if (lobbyWs) {
    lobbyWs.close();
  }
});

// Holt die Liste der aktiven Spiele
const fetchGames = async () => {
  try {
    const response = await fetch('http://127.0.0.1:3000/api/games');
    if (response.ok) {
      gamesList.value = await response.json();
    }
  } catch (error) {
    console.error("Fehler beim Laden der Spiele:", error);
  }
};

// Spiel erstellen
const createGame = async () => {
  try {
    const response = await fetch('http://127.0.0.1:3000/api/games', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ mode: gameMode.value, is_private: isPrivate.value }), 
    });

    if (response.ok) {
      const data = await response.json();
      createdRoomCode.value = data.room_code;
      
      // Hinweis: Wir müssen hier fetchGames() nicht mehr manuell rufen, 
      // da der Server sofort ein 'update' über WebSocket schickt!
      
      if (isPrivate.value && data.password) {
        createdPassword.value = data.password;
        showCreatedPasswordDialog.value = true;
      } else {
        router.push(`/game/${data.room_code}`);
      }
    }
  } catch (error) {
    console.error("Netzwerkfehler:", error);
  }
};

const joinCreatedGame = () => {
  showCreatedPasswordDialog.value = false;
  router.push(`/game/${createdRoomCode.value}`);
};

const initiateJoin = async (game: GameListItem) => {
  joinError.value = '';
  inputPassword.value = '';
  selectedRoom.value = game.room_code;

  if (game.is_private) {
    showJoinDialog.value = true;
  } else {
    await attemptJoin(game.room_code, null);
  }
};

const submitJoin = async () => {
  if (!inputPassword.value) {
    joinError.value = "Passwort wird benötigt";
    return;
  }
  await attemptJoin(selectedRoom.value, inputPassword.value.toUpperCase());
};

const attemptJoin = async (roomCode: string, password: string | null) => {
  try {
    const response = await fetch(`http://127.0.0.1:3000/api/games/${roomCode}/join`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ password }),
    });

    if (response.ok) {
      showJoinDialog.value = false;
      router.push(`/game/${roomCode}`);
    } else if (response.status === 401) {
      joinError.value = "Falsches Passwort!";
    } else {
      joinError.value = "Raum nicht gefunden.";
      fetchGames(); // Falls der Raum in der Zwischenzeit gelöscht wurde
    }
  } catch (error) {
    console.error("Join Fehler:", error);
  }
};
</script>

<style scoped>
.bg-deep-dark { background-color: #121418; background-image: radial-gradient(circle at 50% 0%, #1f2532 0%, #121418 70%); }
.title-glow { text-shadow: 0 0 20px rgba(255, 255, 255, 0.2); letter-spacing: 4px; }
.accent-glow { text-shadow: 0 0 10px currentColor; }
.text-cyan { color: #00e5ff !important; }
.text-orange { color: #ff9800 !important; }

.lobby-wrapper {
  background: linear-gradient(145deg, #252a35, #181a22);
  border-radius: 16px;
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.6), inset 0 1px 2px rgba(255, 255, 255, 0.1), inset 0 -1px 2px rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.05);
  overflow: hidden;
}

@media (min-width: 960px) { .border-right-md { border-right: 1px solid rgba(255, 255, 255, 0.05); } }

.cyber-toggle { background-color: rgba(0, 0, 0, 0.2) !important; border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 8px; width: 100%; }
.cyber-toggle-active { background-color: rgba(0, 229, 255, 0.15) !important; color: #00e5ff !important; border-color: rgba(0, 229, 255, 0.5) !important; box-shadow: inset 0 0 10px rgba(0, 229, 255, 0.2); }

.cyber-btn-primary { color: #00e5ff !important; border-color: rgba(0, 229, 255, 0.3) !important; background-color: rgba(0, 0, 0, 0.2); border-radius: 8px; font-weight: bold; letter-spacing: 1px; }
.cyber-btn-primary:hover { background-color: rgba(0, 229, 255, 0.1); border-color: rgba(0, 229, 255, 0.8) !important; box-shadow: 0 0 15px rgba(0, 229, 255, 0.4); }

/* Server List Area */
.server-list {
  max-height: 350px;
  overflow-y: auto;
  padding-right: 10px;
}

/* Custom Scrollbar */
.server-list::-webkit-scrollbar { width: 6px; }
.server-list::-webkit-scrollbar-track { background: rgba(0,0,0,0.2); border-radius: 4px; }
.server-list::-webkit-scrollbar-thumb { background: rgba(255, 152, 0, 0.5); border-radius: 4px; }

.cyber-server-card {
  background: rgba(0, 0, 0, 0.3) !important;
  border-color: rgba(255, 255, 255, 0.1) !important;
  transition: all 0.2s ease;
}
.cyber-server-card:hover {
  background: rgba(0, 0, 0, 0.5) !important;
  border-color: rgba(255, 152, 0, 0.4) !important;
  transform: translateX(4px);
}

/* Dialogs */
.cyber-dialog {
  border: 1px solid rgba(0, 229, 255, 0.2);
  box-shadow: 0 0 30px rgba(0,0,0,0.8), inset 0 0 20px rgba(0,0,0,0.5);
}
.tracking-widest { letter-spacing: 6px; }

/* Input */
:deep(.cyber-input input) { color: white !important; font-weight: bold; letter-spacing: 2px; text-transform: uppercase; text-align: center; }
</style>