<template>
    <div>
        <b-navbar toggleable="lg" type="dark" variant="success">
            <router-link to="/">
                <b-navbar-brand id="brand"><i class="fas fa-shopping-bag fa-lg"></i> Home</b-navbar-brand>
            </router-link>
            <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

            <b-collapse id="nav-collapse" is-nav>
            <b-navbar-nav>
                <b-nav-item>
                    <router-link class="link" to="/myproducts">My Products</router-link>
                </b-nav-item>        
            </b-navbar-nav>
            <b-navbar-nav class="ml-auto">
                <b-button pill class="mr-3 addProduct" variant="outline-light" size="sm" v-b-modal.modal-add-product><i class="fas fa-plus-circle"></i> Add Product</b-button>
                <b-nav-item-dropdown right>
                <template v-slot:button-content>
                    <em id="user">{{ username }}</em>
                </template>
                <b-dropdown-item><router-link to="/user" id="router-link"><i class="fas fa-user"></i> View profile</router-link></b-dropdown-item>
                <b-dropdown-item><router-link to="/cart" id="router-link"><i class="fas fa-cart-arrow-down"></i> My Cart</router-link></b-dropdown-item>
                <b-dropdown-item @click="logout"><i class="fas fa-sign-out-alt"></i> Sign Out</b-dropdown-item>
                </b-nav-item-dropdown>
            </b-navbar-nav>
            </b-collapse>
        </b-navbar>
         <b-modal id="modal-add-product" title="Add New Product" hide-footer>
            <form>
                <div class="form-group">
                    <label for="inputName">Name</label>
                    <input type="text" class="form-control" id="inputName" placeholder="Product Name" v-model="newName">
                </div>
                <div class="form-group">
                    <label for="inputDescription">Description</label>
                    <textarea class="form-control" id="inputDescription" placeholder="Product Description" rows="2" v-model="newDescription"></textarea>
                </div>
                <div class="form-group">
                    <label for="inputPrice">Price</label>
                    <input class="form-control" type="number" value="0" id="inputPrice" v-model="newPrice">
                </div>
                <div class="form-group">
                    <label for="inputStock">Stock</label>
                    <input class="form-control" type="number" value="0" id="inputStock" v-model="newStock">
                </div>
                <div class="form-group">
                    <label for="inputFile">Image</label>
                    <b-form-file
                        id="inputFile"
                        v-model="newPicture"
                        placeholder="Choose a file or drop it here..."
                        drop-placeholder="Drop file here..."
                    >
                    </b-form-file>
                </div>
                <b-button variant="light" v-on:click.prevent="clear" @click="$bvModal.hide('modal-add-product')">Cancel</b-button>
                <b-button class="ml-4" variant="primary" v-on:click.prevent="addProduct" @click="$bvModal.hide('modal-add-product')">Submit</b-button>
            </form>
        </b-modal>
    </div>
</template>

<script>
import Swal from 'sweetalert2'
import axios from 'axios'
export default {
    name: "navbar",
    data(){
        return {
            username: '',
            newName: '',
            newDescription: '',
            newPrice: null,
            newStock: null,
            newPicture: null
        }
    },
    methods: {
        clear: function(){
            this.newName = '',
            this.newDescription = '',
            this.newPrice = null,
            this.newStock = null,
            this.newPicture = null
        },
        rupiahPrice: function(rawPrice){
            let angka = rawPrice
            let rupiah = '';		
            let numReverse = angka.toString().split('').reverse().join('');
            for(let i = 0; i < numReverse.length; i++) if(i%3 == 0) rupiah += numReverse.substr(i,3)+'.';
            return 'Rp. '+rupiah.split('',rupiah.length-1).reverse().join('');
        },
        addProduct: function(){
            this.$emit('nowLoading')
            let formData = new FormData()
            formData.append('name', this.newName)
            formData.append('description', this.newDescription)
            formData.append('price', this.newPrice)
            formData.append('stock', this.newStock)
            formData.append('picture', this.newPicture)
            axios({
                method: 'post',
                url: 'http://ecommerce-server.kennys.my.id:3000/products',
                data: formData,
                headers: {
                    token: localStorage.getItem('token')
                }
            })
            .then(({data})=>{
                let detail = `Price: ${this.rupiahPrice(data.price)}<br>Stock: ${data.stock}`
                this.newName = '',
                this.newDescription = '',
                this.newPrice = null,
                this.newStock = null,
                this.newPicture = null
                this.$emit('productAdded')
                Swal.fire({
                    icon: 'success',
                    title: `Product "${data.name}" Added!`,
                    html: detail,
                })
            })
            .catch(()=>{
                Swal.fire({
                    icon: 'error',
                    title: 'Error'
                })
            })
        },
        logout: function(){
            localStorage.clear()
            this.$router.push('/loginregister')
            Swal.fire(
                'You are logged out!',
                '',
                'success'
            )
        }
    },
    created(){
        this.username = localStorage.getItem('username')
    }
}
</script>

<style scoped>
.addProduct {
    font-weight: bold;
}

#router-link {
    text-decoration: none;
    color: black;
}

.link {
    text-decoration: none;
    color: white;
    filter: opacity(80%);
    font-weight: lighter;
    font-size: 20px;
}
.link:hover {
    filter: opacity(100%)
}
#brand {
    font-weight: bold;
}
#user {
    font-size: 19px;
    font-weight: bold
}
</style>