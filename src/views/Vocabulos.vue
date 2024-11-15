<template>
    <div class="about">
      <h1>Gerenciar Vocabulos</h1>
      <p>
        <label for="termo">Termo: </label>
        <input id="termo" type="text" v-model="vocabulo.termo" />
      </p>
      <p>
        <label for="significado">Significado: </label>
        <input id="significado" type="text" v-model="vocabulo.significado" />
      </p>
      <p>
        <label for="versao">Versão: </label>
        <input id="versao" type="number" v-model="vocabulo.versao" />
      </p>
      <p>
        <label for="situacao">Data de Desativação: </label>
        <input
          id="situacao"
          type="datetime-local"
          v-model="vocabulo.dataHoraDesativacao"
          placeholder="Opcional"
        />
      </p>
      <button @click="incluir">Incluir</button>
      <button @click="consultar">Consultar</button>
      <button @click="buscarVocabulos">Listar Todos</button>
      <p class="error">{{ erro }}</p>
      <table v-if="Vocabulos.length > 0">
        <thead>
          <tr>
            <th>Id</th>
            <th>Termo</th>
            <th>Significado</th>
            <th>Versão</th>
            <th>Situação</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="vocabulo in Vocabulos"
            :key="vocabulo.id"
            :class="vocabulo.dataHoraDesativacao ? 'desativado' : 'ativo'"
          >
            <td>{{ vocabulo.id }}</td>
            <td>{{ vocabulo.termo }}</td>
            <td>{{ vocabulo.significado }}</td>
            <td>{{ vocabulo.versao }}</td>
            <td>{{ vocabulo.dataHoraDesativacao ? 'Desativado' : 'Ativo' }}</td>
          </tr>
        </tbody>
      </table>
      <p v-else>Nenhum vocábulo foi encontrado para os critérios fornecidos.</p>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, onMounted } from 'vue';
  import axios from 'axios';
  
  // Definindo uma interface para o vocábulo
  interface Vocabulo {
    id: string;
    termo: string;
    significado: string;
    versao: number;
    dataHoraCadastro: string;
    dataHoraDesativacao: string | null;
  }
  
  const vocabulo = ref<Vocabulo>({
    id: '',
    termo: '',
    significado: '',
    versao: 0,
    dataHoraCadastro: '',
    dataHoraDesativacao: null,
  });
  
  const Vocabulos = ref<Vocabulo[]>([]); // Especificando que Vocabulos é um array de objetos Vocabulo
  const erro = ref("");
  
  function validarCampos() {
    if (!vocabulo.value.termo || !vocabulo.value.significado || !vocabulo.value.versao) {
      erro.value = "Por favor, preencha todos os campos obrigatórios.";
      return false;
    }
    return true;
  }
  
  async function incluir() {
    erro.value = "";
  
    if (!validarCampos()) return;
  
    // Garante que dataHoraDesativacao será null se o campo estiver vazio
    if (!vocabulo.value.dataHoraDesativacao) {
      vocabulo.value.dataHoraDesativacao = null;
    }
  
    try {
      await axios.post("vocabulo", vocabulo.value);
      await buscarVocabulos();
      limparCampos();
    } catch (e) {
      erro.value = (e as Error).message;
    }
  }
  
  async function buscarVocabulos() {
    try {
      const response = await axios.get("vocabulo");
      Vocabulos.value = response.data;
    } catch (e) {
      erro.value = (e as Error).message;
    }
  }
  
  async function consultar() {
    erro.value = "";
  
    if (!vocabulo.value.termo || !vocabulo.value.versao) {
      erro.value = "Por favor, preencha os campos 'termo' e 'versão' para a consulta.";
      return;
    }
  
    try {
      const response = await axios.get(
        `vocabulo/${vocabulo.value.termo}/${vocabulo.value.versao}`
      );
      Vocabulos.value = response.data;
  
      // Mostra mensagem se nenhum vocábulo for encontrado
      if (Vocabulos.value.length === 0) {
        erro.value = "Nenhum vocábulo foi encontrado para os critérios fornecidos.";
      }
    } catch (e) {
      erro.value = "Erro ao consultar vocábulo: " + (e as Error).message;
      Vocabulos.value = [];
    }
  }
  
  function limparCampos() {
    vocabulo.value = {
      id: '',
      termo: '',
      significado: '',
      versao: 0,
      dataHoraCadastro: '',
      dataHoraDesativacao: null,
    };
  }
  
  onMounted(() => {
    buscarVocabulos();
  });
  </script>
  
  <style>
  tr.desativado {
    background-color: #4d0006;
  }
  
  tr.ativo {
    background-color: #00300b;
  }
  
  .error {
    color: red;
    font-weight: bold;
  }
  </style>
  