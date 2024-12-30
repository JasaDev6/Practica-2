<template>
  <div>
    <h2>Productos</h2>
    <div>
      <input v-model="search" placeholder="Buscar por nombre..." @input="buscarProductos" />
      <select v-model="priceFilter" @change="filtrarPorPrecio">
        <option value="0">Filtrar por precio</option>
        <option value="100">Mayor a 100</option>
      </select>
    </div>
    
    <table>
      <thead>
        <tr>
          <th>Nombre</th>
          <th>Precio</th>
          <th>Categoría</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="producto in productos" :key="producto.id">
          <td>{{ producto.nombre }}</td>
          <td>{{ producto.precio }}</td>
          <td>{{ getCategoriaName(producto.categoriaId) }}</td>
          <td>
            <button @click="editarProducto(producto.id)">Editar</button>
            <button @click="eliminarProducto(producto.id)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Formulario para agregar/editar productos -->
    <div>
      <h3>{{ editMode ? 'Editar Producto' : 'Agregar Producto' }}</h3>
      <form @submit.prevent="submitForm">
        <input v-model="producto.nombre" placeholder="Nombre" required />
        <input v-model="producto.precio" type="number" placeholder="Precio" required />
        <select v-model="producto.categoriaId" required>
          <option v-for="categoria in categorias" :key="categoria.id" :value="categoria.id">
            {{ categoria.nombre }}
          </option>
        </select>
        <button type="submit">{{ editMode ? 'Actualizar' : 'Agregar' }}</button>
      </form>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      productos: [],
      categorias: [],
      producto: { nombre: '', precio: 0, categoriaId: null },
      editMode: false,
      search: '',
      priceFilter: '0',
    };
  },
  methods: {
    async fetchProductos() {
      try {
        const response = await axios.get('http://localhost:5000/productos');
        this.productos = response.data;
      } catch (error) {
        console.error('Error fetching productos:', error);
      }
    },
    async fetchCategorias() {
      try {
        const response = await axios.get('http://localhost:5000/categorias');
        this.categorias = response.data;
      } catch (error) {
        console.error('Error fetching categorias:', error);
      }
    },
    async crearProducto() {
      try {
        const response = await axios.post('http://localhost:5000/productos', this.producto);
        this.productos.push(response.data);
        this.resetForm();
      } catch (error) {
        console.error('Error creating producto:', error);
      }
    },
    async editarProducto(id) {
      const producto = this.productos.find(p => p.id === id);
      this.producto = { ...producto };
      this.editMode = true;
    },
    async actualizarProducto() {
      try {
        const response = await axios.put(`http://localhost:5000/productos/${this.producto.id}`, this.producto);
        const index = this.productos.findIndex(p => p.id === this.producto.id);
        this.productos[index] = response.data;
        this.resetForm();
      } catch (error) {
        console.error('Error updating producto:', error);
      }
    },
    async eliminarProducto(id) {
      try {
        await axios.delete(`http://localhost:5000/productos/${id}`);
        this.productos = this.productos.filter(p => p.id !== id);
      } catch (error) {
        console.error('Error deleting producto:', error);
      }
    },
    resetForm() {
      this.producto = { nombre: '', precio: 0, categoriaId: null };
      this.editMode = false;
    },
    submitForm() {
      if (this.editMode) {
        this.actualizarProducto();
      } else {
        this.crearProducto();
      }
    },
    buscarProductos() {
      axios.get(`http://localhost:5000/productos?q=${this.search}`)
        .then(response => {
          this.productos = response.data;
        });
    },
    filtrarPorPrecio() {
      if (this.priceFilter === '100') {
        axios.get('http://localhost:5000/productos?precio_gte=100')
          .then(response => {
            this.productos = response.data;
          });
      } else {
        this.fetchProductos();
      }
    },
    getCategoriaName(id) {
      const categoria = this.categorias.find(c => c.id === id);
      return categoria ? categoria.nombre : '';
    },
  },
  created() {
    this.fetchProductos();
    this.fetchCategorias();
  },
};
</script>

<style scoped>
/* Agrega tus estilos aquí */
</style>
