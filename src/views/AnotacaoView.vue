<template>
   <div>
    <h1>Gerenciar Anotações</h1>
    <p>Texto: <input type="text" v-model="texto"/></p>
    <p>Antiguidade: <input type="datetime-local" v-model="dataHora"/></p>
    <p>
        Usuário:
        <select name="usuario" id="usuario" v-model="idUsuario">
            <option v-for="(usuario, index) in usuarios" :key="index" :value="usuario.id">
                {{ usuario.nome }}
            </option>
        </select>
    </p>
    <button @click="registerAnotacao">Cadastrar</button>
    <p v-if="erro"> {{ erro }}</p>
    <p>Pesquisar Texto: <input type="text" v-model="textoPesquisa"/></p>
    <p>
        Usuário:
        <select name="usuarioPesquisa" id="usuarioPesquisa" v-model="nomeUsuario">
            <option v-for="(usuario, index) in usuarios" :key="index" :value="usuario.nome">
                {{ usuario.nome }}
            </option>
        </select>
    </p>
    <button @click="search">Pesquisar</button>
    <table v-if="anotacoes.length > 0">
        <thead>
            <tr>
                <td>Id</td>
                <td>Texto</td>
                <td>Antiguidade</td>
                <td>Nome</td>
            </tr>
        </thead>
        <tbody>
            <tr v-for="(anotacao, index) in anotacoes" :key="index">
                <td> {{ anotacao.id }}</td>
                <td> {{ anotacao.texto }}</td>
                <td> {{ qntMeses(new Date(anotacao.dataHora)) }} meses </td>
                <td> {{ anotacao.usuario.nome }}</td>
            </tr>
        </tbody> 
    </table>
    <p v-else-if="textoPesquisa || nomeUsuario"> “Nenhum registro foi encontrado para os critérios fornecidos” </p>
   </div> 
</template>

<script setup lang="ts">
import axios from 'axios';
import { onMounted, ref, toValue } from 'vue';


const idUsuario = ref<number>();
const texto = ref<string>('');
const nomeUsuario = ref<string>('');
const dataHora = ref<Date>();
const erro = ref<string>('');
const anotacoes = ref<Anotacao[]>([]);
const usuarios = ref<Usuario[]>([]);
const textoPesquisa = ref<string>('');


interface Anotacao{
    id: number,
    texto: string,
    dataHora: string,
    usuario: Usuario
}

interface Usuario{
    id: number
    nome: string
}


async function fetchAnotacao() {
    try{
        anotacoes.value = (await axios.get<Anotacao[]>('anotacao')).data
        erro.value = ''
    } catch (e) {
        erro.value = (e as Error).message
    }
}

async function fetchUsuario() {
    try{
        usuarios.value = (await axios.get<Usuario[]>('usuario')).data
        erro.value = ''
    } catch (e){
        erro.value = (e as Error).message
    }
    
}

async function registerAnotacao() {
    if(texto.value && idUsuario.value){
        try{
            await axios.post('anotacao', {
                texto: texto.value,
                dataHora: dataHora.value,
                usuario: {
                    id: idUsuario.value,
                },
            })
            erro.value = ''
            texto.value = ''
            dataHora.value = undefined
            idUsuario.value = undefined
            fetchAnotacao()
        } catch(e){
            erro.value = (e as Error).message
        }
    } else {
        erro.value = 'Texto ou usuário vazios!'
    }
}

async function search() {
    try{
        anotacoes.value = (await axios.get('anotacao/' + nomeUsuario.value + '/' + textoPesquisa.value )).data
        erro.value = ''
    } catch (e){
        erro.value = (e as Error).message
    }
}

function qntMeses(date1 : Date): number {
    const date2: Date = new Date()
    return (date2.getFullYear() - date1.getFullYear()) * 12 + (date2.getMonth() - date1.getMonth())
} 

onMounted(() => {
    fetchAnotacao()
    fetchUsuario()
})
</script>