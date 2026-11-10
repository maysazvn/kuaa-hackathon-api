<script setup>
import { ref, computed } from 'vue'
const emit = defineEmits(['fechar', 'adicionar'])
import { postagens } from '@/data/postagens.js'
import { salas } from '@/data/salas.js'
import { loginOut } from '@/views/account/login/Loginout'
import { salasUsuario } from '@/data/salasUsuario'
import { pegarIDUsuario } from '@/views/account/login/UserReal'

const postagensTituloNovo = ref('')
const postagensConteudoNovo = ref('')
const salaSelecionada = ref(null)
const salaFinal = computed(() => salaSelecionada.value)
const storage = 'postagens'
const imagemPost = ref('')
const API_URL = 'http://localhost:3000'; 

function usuarioEstaNaSala(salaIdDoPost) {
  if (!Array.isArray(salasUsuario.value)) return false

  console.log(
    'Testando sala do sistema:',
    salaIdDoPost,
    'Lista do usuário:',
    JSON.parse(JSON.stringify(salasUsuario.value)),
  )

  for (let sala of salasUsuario.value) {
    const idUsuario = sala.idSala || sala.id

    if (Number(idUsuario) === Number(salaIdDoPost)) {
      return true
    }
  }
  return false
}

async function adicionar() {
  if (
    !postagensTituloNovo.value.trim() ||
    !postagensConteudoNovo.value.trim() ||
    !salaFinal.value
  ) {
    alert(`Preencha os campos!`)
  } else {
    let maiorId = Math.max(...postagens.value.map((item) => item.id))
    const novoPost = {
      titulo: postagensTituloNovo.value,
      conteudo: postagensConteudoNovo.value,
      autorID: pegarIDUsuario(),
      data: new Date().toLocaleDateString('pt-BR'),
      id: maiorId + 1,
      salaId: Number(salaFinal.value),
      imagem: imagemPost.value || '',
      curtidas: 0,
      salvos: 0,
    }

    console.log(`sala selecionada: ${salaFinal.value}`)


await fetch(`${API_URL}/postagens`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify(novoPost.value)
});

    postagens.value.unshift(novoPost)
    postagensTituloNovo.value = ''
    postagensConteudoNovo.value = ''
    imagemPost.value = ''
    console.log('Post atualizado:', JSON.parse(JSON.stringify(postagens.value)))

    localStorage.setItem(storage, JSON.stringify(postagens.value))
    emit('fechar')
  }
}
</script>
<template>
  <span v-if="loginOut === 'ativo'">
    <section class="sessaoPostar">
      <v-card theme="dark" class="caixa">
        <h2 class="titulo-postar">Postar</h2>

        <v-card-text class="conteudo-postar">
          <v-row density="comfortable">
            <v-col cols="12" md="12">
              <v-text-field
                label="Título *"
                required
                v-model="postagensTituloNovo"
                class="formulario"
                variant="outlined"
                base-color="#3e3e3e"
                bg-color="#1e1e1e"
              ></v-text-field>
            </v-col>

            <v-col cols="12" md="12">
              <v-textarea
                hint="Compartilhe suas ideias"
                label="O que está pensando agora? *"
                v-model="postagensConteudoNovo"
                class="formulario"
                variant="outlined"
                base-color="#3e3e3e"
                bg-color="#1e1e1e"
                rows="3"
              ></v-textarea>
            </v-col>

            <v-col cols="12" md="12">
              <v-textarea
                hint="Imagem"
                label="Uma imagem pode dizer mais que palavras (URL imagem)"
                v-model="imagemPost"
                class="formulario"
                variant="outlined"
                base-color="#3e3e3e"
                bg-color="#1e1e1e"
                rows="3"
              ></v-textarea>
            </v-col>

            <v-col cols="12">
              <v-autocomplete
                v-model="salaSelecionada"
                :items="(salas.value || salas).filter((sala) => usuarioEstaNaSala(sala.idSala))"
                item-title="nome"
                item-value="idSala"
                label="Sala *"
                required
                auto-select-first
                class="formulario"
                variant="outlined"
                base-color="#3e3e3e"
                bg-color="#1e1e1e"
                :menu-props="{ contentClass: 'menu-salas-custom' }"
                :no-data-text="'Sem salas'"
              ></v-autocomplete>
            </v-col>
          </v-row>

          <small class="aviso-obrigatorio"> * Indica campo obrigatório </small>
        </v-card-text>

        <v-divider class="divisor"></v-divider>

        <v-card-actions class="acoes">
          <button class="btnPostar" @click="adicionar">Postar</button>
          <button class="btnFechar" @click="$emit('fechar')">Fechar</button>
        </v-card-actions>
      </v-card>
    </section>
  </span>
  <span v-else-if="loginOut === 'inativo'" class="login">
    <p>Faça Login para criar posts!</p>
  </span>
</template>

<style scoped>
.caixa {
  padding: 25px;
  border-radius: 20px !important;
  background-color: #262626 !important;
  color: #d9d9d9 !important;
}

.titulo-postar {
  font-size: 1.8rem;
  font-weight: bold;
  margin-bottom: 20px;
}

.conteudo-postar {
  padding: 0 !important;
}

:deep(.formulario .v-field) {
  border-radius: 15px !important;
  padding-left: 15px !important;
  padding-right: 15px !important;
}

:deep(.formulario .v-field__input) {
  padding-top: 15px !important;
  padding-bottom: 15px !important;
  font-size: 1rem;
}

:deep(.formulario.v-input--focused .v-field__outline) {
  --v-field-border-width: 2px;
}

:deep(.formulario input),
:deep(.formulario textarea) {
  color: #d9d9d9 !important;
}

:deep(.formulario .v-chip) {
  border-radius: 8px !important;
}

.aviso-obrigatorio {
  color: #888888;
  font-size: 0.8rem;
  display: block;
  margin-top: 10px;
}

.divisor {
  border-color: #747474 !important;
  margin: 15px 0;
}

.acoes {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 0;
}

.btnPostar {
  background-color: #f8d76b;
  color: #1e1e1e;
  font-weight: bold;
  border-radius: 10px;
  padding: 8px 20px;
  border: none;
  cursor: pointer;
  transition: 0.2s;
}

.btnPostar:hover {
  opacity: 0.9;
  transform: scale(0.96);
}

.btnFechar {
  background-color: #3e3e3e;
  color: #d9d9d9;
  font-weight: bold;
  border-radius: 10px;
  padding: 8px 20px;
  border: none;
  cursor: pointer;
  transition: 0.2s;
}

.btnFechar:hover {
  opacity: 0.9;
  transform: scale(0.96);
}

.login {
  text-align: center;
  color: #d9d9d9;
  font-size: 2rem;
  font-weight: bold;
  background-color: #1e1e1e;
  border-radius: 20px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.163);
}
</style>

<style>
.menu-salas-custom .v-list-item-title {
  margin-left: 14px !important;
}

.menu-salas-custom .v-list-item {
  padding: 8px 12px !important;
}
</style>
