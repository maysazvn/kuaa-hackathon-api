<script setup>
import { ref, onMounted, watch } from 'vue'
import { loginOut } from '@/views/account/login/Loginout'
import { userReal } from '@/views/account/login/UserReal'

const API_URL = 'http://localhost:3000'

const props = defineProps({
  sala: {
    type: Object,
    required: true
  }
})


const emit = defineEmits(['atualizar-participantes'])

const participando = ref(false)
const carregando = ref(false)

async function verificarParticipacao() {
  if (!userReal.value) {
    participando.value = false
    return
  }

  try {
    const resposta = await fetch(
      `${API_URL}/salasUsuario?salaId=${props.sala.id}&usuario=${encodeURIComponent(userReal.value)}`
    )
    const participacoes = await resposta.json()
    participando.value = participacoes.length > 0
  } catch (erro) {
    console.error('Erro ao verificar participação:', erro)
  }
}

async function entrar() {
  if (loginOut.value !== 'ativo') {
    alert('Faça login para entrar!')
    return
  }

  if (carregando.value || participando.value) return
  carregando.value = true

  try {
    await verificarParticipacao()
    if (participando.value) return


    const resposta = await fetch(`${API_URL}/salasUsuario`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        salaId: String(props.sala.id),
        usuario: userReal.value
      })
    })

    if (!resposta.ok) throw new Error('Erro ao entrar na sala')


    const resSala = await fetch(`${API_URL}/salas/${props.sala.id}`)
    const salaAtual = await resSala.json()
    const novaQuantidade = (Number(salaAtual.participantes) || 0) + 1


    const atualizacao = await fetch(`${API_URL}/salas/${props.sala.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        participantes: novaQuantidade
      })
    })

    if (!atualizacao.ok) throw new Error('Erro ao atualizar participantes')

    props.sala.participantes = novaQuantidade
    emit('atualizar-participantes', novaQuantidade)
    participando.value = true

  } catch (erro) {
    console.error(erro)
    alert('Não foi possível entrar na sala.')
    await verificarParticipacao()
  } finally {
    carregando.value = false
  }
}

async function sair() {
  if (carregando.value) return
  carregando.value = true

  try {
    const resposta = await fetch(
      `${API_URL}/salasUsuario?salaId=${props.sala.id}&usuario=${encodeURIComponent(userReal.value)}`
    )
    const participacoes = await resposta.json()

    if (participacoes.length === 0) {
      participando.value = false
      return
    }

    for (const participacao of participacoes) {
      const delRes = await fetch(`${API_URL}/salasUsuario/${participacao.id}`, {
        method: 'DELETE'
      })
      if (!delRes.ok) throw new Error('Erro ao remover vínculo')
    }

    const resSala = await fetch(`${API_URL}/salas/${props.sala.id}`)
    const salaAtual = await resSala.json()
    const novaQuantidade = Math.max(0, (Number(salaAtual.participantes) || 0) - 1)

    const atualizacao = await fetch(`${API_URL}/salas/${props.sala.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        participantes: novaQuantidade
      })
    })

    if (!atualizacao.ok) throw new Error('Erro ao atualizar participantes ao sair')

    props.sala.participantes = novaQuantidade
    emit('atualizar-participantes', novaQuantidade)
    participando.value = false

  } catch (erro) {
    console.error(erro)
    alert('Não foi possível sair da sala.')
    await verificarParticipacao()
  } finally {
    carregando.value = false
  }
}

onMounted(verificarParticipacao)
watch(userReal, verificarParticipacao)
</script>

<template>
  <button
    v-if="participando"
    @click="sair"
    :disabled="carregando"
    class="btn-sair"
  >
    {{ carregando ? 'Aguarde...' : 'Sair' }}
  </button>

  <button
    v-else
    @click="entrar"
    :disabled="carregando"
    class="btn-entrar"
  >
    {{ carregando ? 'Aguarde...' : 'Entrar' }}
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