<script setup>
import { ref, onMounted, computed } from 'vue'
import { pegarIDUsuario } from '@/views/account/login/UserReal'
import { users } from '@/views/user/Users'
import { comentariosReais } from '@/data/comentarios'
import { loginOut } from '@/views/account/login/Loginout'

const props = defineProps(['post', 'usuario'])

let comentarios = ref(null)
const novoComent = ref({
  postId: String(props.post.id),
  texto: '',
  data: '',
  autorID: '',
  curtido: false,
  curtidas: 0,
})

const usuarioLogado = computed(() => pegarIDUsuario())


const API_URL= 'http://localhost:3000'

async function CarregarComentarios() {
  const respostaComentararios = await fetch(`${API_URL}/comentarios`);
  const todosComentarios = await respostaComentararios.json();

 comentarios.value = todosComentarios.filter(
    (c) => String(c.postId) === String(props.post.id)
  )}


function pegarNomeAutor(autorID) {
  const usuarioEncontrado = users.find((usu) => String(usu.id) === String(autorID))
  if (usuarioEncontrado) {
    return usuarioEncontrado.nome
  } else {
    return 'Usuário desconhecido'
  }
}

function pegarFotoUsuario(autorID) {
  const usuarioEncontrado = users.find((usu) => String(usu.id) === String(autorID))
  if (usuarioEncontrado) {
    return usuarioEncontrado.pfp
  } else {
    return '/pfpPlaceholder.png'
  }
}

async function comentar() {
  if (loginOut.value == 'ativo') {
    if (!novoComent.value.texto.trim()) {
      alert('Preencha os campos!!')
    } else {
      novoComent.value.postId = String(props.post.id)
      novoComent.value.data = new Date().toLocaleString('pt-BR')
      novoComent.value.autorID = String(pegarIDUsuario())
      novoComent.value.texto = novoComent.value.texto.trim()

      const resposta = await fetch(`${API_URL}/comentarios`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(novoComent.value)
      })

      const comentarioSalvo = await resposta.json()
      comentarios.value.unshift(comentarioSalvo)
      novoComent.value.texto = ''
    }
  } else {
    alert('Faça login para comentar!')
  }
}





async function CurtidaCOemn(comentario) {
  if (loginOut.value === 'ativo') {
    if (comentario.curtido === true) {
      comentario.curtido = false
      comentario.curtidas--
    } else {
      comentario.curtido = true
      comentario.curtidas++
    }
   await fetch(`${API_URL}/comentarios/${comentario.id}`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        curtido: comentario.curtido,
        curtidas: comentario.curtidas
      })
    })
  } else {
    alert('Faça login para curtir!')
  }
}

async function excluir(id) {
  const confirmarExcluir = window.confirm('Você tem certeza que deseja excluir esse comentário? essa ação não tem volta')

  if (confirmarExcluir) {
    await fetch(`${API_URL}/comentarios/${id}`, {
      method: 'DELETE'
    });
    await CarregarComentarios()
  }
}

async function editar(comentario) {
  const edicao = prompt(`Edite o seu comentário`, comentario.texto)

  if (edicao !== null && edicao.trim() !== '') {
    comentario.texto = edicao
    await fetch(`${API_URL}/comentarios/${comentario.id}`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json'
      }, body: JSON.stringify({
        texto: edicao.trim()
      })
    })
    comentario.texto = edicao.trim()
  }
}



// function editar(comentario) {
//   const index = comentarios.value.findIndex((c) => c.id === comentario.id)

//   if (index !== -1) {
//     const edicao = prompt(`Edite o seu comentario...`, comentario.texto)

//     if (edicao !== null) {
//       comentarios.value[index].texto = edicao
//       localStorage.setItem(chaveStorage, JSON.stringify(comentarios.value))
//     }
//   }
// }
// ////

function mostrarItens(comentario) {
  comentario.aberto = !comentario.aberto
}

function denunciar() {
  if(loginOut.value == 'ativo'){
    alert('Comentário denunciado com sucesso.')
  }else{
    alert('Faça login para denunciar!')
  }
}


onMounted(() => {
  CarregarComentarios()
})


</script>
<template>
  <section>
    <div class="comentarios">
      <div class="comentar">
        <textarea placeholder="O que está pensando?" v-model="novoComent.texto"></textarea>
      </div>
      <!-- aquiiiiiiiiiiiiiiiiiiiiiiiiiiiiiii -->

        <button type="submit" @click="comentar()" class="comentarBtn">Comentar</button>




      <div class="todos" v-for="comentario in comentarios" :key="comentario.id">
        <div class="cima">
          <p class="autor">
            <strong>
              <RouterLink :to="`/otherProfile/${comentario.autorID}`">
                <img :src="pegarFotoUsuario(comentario.autorID)" class="fotoAutor" />
              </RouterLink>
              <RouterLink :to="`/otherProfile/${comentario.autorID}`">
                {{ pegarNomeAutor(comentario.autorID) }}
              </RouterLink>
            </strong>
          </p>
          <button class="editarDeletar" v-on:click="mostrarItens(comentario)">•••</button>

          <div class="vshow" v-show="comentario.aberto">
            <div v-if="String(comentario.autorID) === String(usuarioLogado)" class="btnsEditarDeletar">
              <button @click="editar(comentario)" class="editar">Editar</button>
              <button @click="excluir(comentario.id)" class="deletar">Excluir</button>
            </div>
            <!-- aquiiiiiiiiiiiiii -->

            <div v-else>
              <button @click="denunciar()" class="denunciar">Denunciar</button>

            </div>
          </div>
        </div>
        <p class="texto">{{ comentario.texto }}</p>

        <div class="interacao">
          <div class="curtidas">
            <button @click="CurtidaCOemn(comentario)" class="mostrarComent"><font-awesome-icon icon="heart" :style="{color: comentario.curtido ? 'red' : 'grey'}" /> <span>{{ comentario.curtidas }}</span></button>
          </div>


        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
.comentarios {
  border-top: 2px solid #3e3e3e;
}

textarea {
  border: 2px solid #3e3e3e;
  width: 450px !important;
  height: 90px !important;
  border-radius: 10px;
  margin-top: 25px;
  resize: none;
  outline: none;
  padding: 5px 10px;
}

.cima {
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: relative;
}

textarea:focus {
  border: 2px solid #f8d76bb8;
}

div.todos {
  margin-bottom: 20px;
  margin-top: 20px;
  border-bottom: 1px solid #3e3e3e;
  padding: 5px;
  padding-bottom: 20px;
}

h2.usuario {
  margin: 0;
  font-size: 1.2rem;
  font-weight: bold;
}

p.texto {
  overflow-wrap: break-word;
  max-width: 30vw;
  margin: 2px;
}

button.comentarBtn {
  background: #f8d76b;
  color: #1e1e1e;
  font-weight: bold;
  border-radius: 10px;
  padding: 5px 15px;
  margin: 5px 0;
}

button.comentarBtn:hover {
  opacity: 0.9;
  transform: scale(0.95);
  transition: 0.2s;
}

button.editarDeletar {
  background: transparent;
  border: none;
  color: inherit;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 2px 8px;
}

.vshow {
  position: absolute;
  top: 100%;
  right: 0;
  background-color: #313131;
  color: #d9d9d9;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.179);
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  padding: 5px;
  z-index: 10;
  font-size: 0.8rem;
}

.btnsEditarDeletar {
  display: flex;
  flex-direction: column;
}

.editar:hover,
.deletar:hover,
.denunciar:hover {
  color: #f8d668;
  transform: scale(0.97);
  transition: 0.3s;
}

p.autor {
  margin: 0;
  display: flex;
  flex-direction: column;
}

p.autor a {
  font-size: 1rem;
  font-weight: 700;
  text-decoration: none;
}

.fotoAutor {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
  vertical-align: middle;
  margin-right: 6px;
}

a:hover {
  opacity: 0.6;
  transition: 0.4s;
}

.interacao {
  display: flex;
  gap: 10px;
  margin: 2px 2px 7px 2px;
}

button.mostrarComent {
  background-color: #3e3e3e;
  border-radius: 25px;
  padding: 6px 15px;
  margin-top: 10px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: bold;
}

button:hover {
  opacity: 0.9;
  transform: scale(0.95);
  transition: 0.2s;
}

@media (max-width: 768px) {
  .comentarios {
    border-top: 2px solid #3e3e3e;
  }

  textarea {
    border: 2px solid #3e3e3e;
    width: 100% !important;
    height: 90px !important;
    border-radius: 10px;
    margin-top: 25px;
    resize: none;
    outline: none;
    padding: 5px 10px;
  }

  p.texto{
    max-width: 100%;
  }
}
</style>
