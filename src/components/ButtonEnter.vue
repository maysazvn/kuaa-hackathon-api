<script setup>
import { ref, onMounted } from 'vue'
import { loginOut } from '@/views/account/login/Loginout'
import { userReal } from '@/views/account/login/UserReal'

const API_URL = 'http://localhost:3000'

const props = defineProps({
  sala: {
    type: Object,
    required: true
  }
})

const participando = ref(false)
const carregando = ref(false)

async function verificarParticipacao() {
  const resposta = await fetch(
    `${API_URL}/salasUsuario?salaId=${props.sala.id}&usuario=${userReal.value}`
  )

  const participacoes = await resposta.json()
  participando.value = participacoes.length > 0
}

async function entrar() {
  if (loginOut.value !== 'ativo') {
    alert('Faça login para entrar!')
    return
  }

  if (carregando.value) return
  carregando.value = true

  try {
    const resposta = await fetch(`${API_URL}/salasUsuario`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        salaId: props.sala.id,
        usuario: userReal.value
      })
    })

    if (!resposta.ok) throw new Error('Erro ao entrar na sala')

    await fetch(`${API_URL}/salas/${props.sala.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        participantes: Number(props.sala.participantes) + 1
      })
    })

    props.sala.participantes++
    participando.value = true
  } catch (erro) {
    alert('Não foi possível entrar na sala.')
    console.error(erro)
  } finally {
    carregando.value = false
  }
}

async function sair() {
  if (carregando.value) return
  carregando.value = true

  try {
    const busca = await fetch(
      `${API_URL}/salasUsuario?salaId=${props.sala.id}&usuario=${userReal.value}`
    )

    const participacoes = await busca.json()

    if (participacoes.length > 0) {
      await fetch(`${API_URL}/salasUsuario/${participacoes[0].id}`, {
        method: 'DELETE'
      })

      await fetch(`${API_URL}/salas/${props.sala.id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          participantes: Math.max(0, Number(props.sala.participantes) - 1)
        })
      })

      props.sala.participantes = Math.max(
        0,
        Number(props.sala.participantes) - 1
      )

      participando.value = false
    }
  } catch (erro) {
    alert('Não foi possível sair da sala.')
    console.error(erro)
  } finally {
    carregando.value = false
  }
}

onMounted(verificarParticipacao)
</script>

<template>
  <button v-if="participando" @click="sair" :disabled="carregando">
    Sair
  </button>

  <button v-else @click="entrar" :disabled="carregando">
    Entrar
  </button>
</template>

<style scoped>
.btn-entrar,
.btn-sair {
  font-weight: bold;
  font-size: 0.95rem;
  padding: 5px 15px;
  border-radius: 25px;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.btn-entrar {
  background: #F8D76B;
  color: #1e1e1e;
}

.btn-sair {
  background-color: #3c3c3c;
  color: #d9d9d9;
}

.btn-sair:hover {
  background-color: #e53935;
  color: #ffffff;
}

button:hover {
  opacity: 0.9;
  transform: scale(0.95);
}
</style>