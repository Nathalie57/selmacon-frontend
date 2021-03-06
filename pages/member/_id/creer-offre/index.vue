<template>
    <div class="table-container" v-if="username && user && id==user.id">
        <ul class="member-menu">
            <li class="orange-text"><router-link tag="a" :to="{name:'member-id-profil', params: { id:user.id }}" exact>Mon profil</router-link></li>
            <li class="blue-text"><router-link tag="a" :to="{name:'member-id-dashboard', params: { id:user.id }}" exact>Gérer mes offres et demandes en cours</router-link></li>
            <li class="blue-text"><router-link tag="a" :to="{name:'member-id-echanges', params: { id:user.id }}" exact>Gérer mes échanges</router-link></li>
            <li class="orange-text"><router-link tag="a" :to="{name:'member-id-creer-offre', params: { id:user.id }}" exact>Déposer une offre</router-link></li>
            <li class="green-text"><router-link tag="a" :to="{name:'member-id-creer-demande', params: { id:user.id }}" exact>Déposer une demande</router-link></li>
            <li class="orange-text"><router-link tag="a" :to="{name:'member-id-membres', params: { id:user.id }}" exact>Liste des membres</router-link></li>
        </ul>
        <h2>Bonjour {{ user.username }} !</h2>
        <div class="form-container">
            <h2>Quelle offre souhaitez-vous déposer ?</h2>
            <form method="POST" @submit.prevent="newOffer">
                <p>
                    <label>Titre de l'offre (choisissez un titre suffisamment explicite) :</label><br/>
                    <input type="text" v-model="title" required autofocus>
                </p>
                <p>
                    <label>Description de l'offre (soyez le plus précis possible) :</label><br/>
                    <textarea v-model="description" required></textarea>
                </p>  
                <p>
                    <label>Catégorie de l'offre</label>                
                    <select v-model="category">
                        <option v-for="category in categories" :value="category.id">
                        {{ category.title }}
                        </option>
                    </select>
                </p>
                <p>
                    <label>Date d'expiration de l'offre (maximum un mois)</label>
                    <input type="date" v-model="expirationDate" required>
                </p>
                <button type="submit">Valider</button>
            </form>  
        </div>
    </div>
    <div v-else class="table-container">
        <p>Vous n'êtes pas autorisé à voir cette page. Veuillez vous connecter.</p>
    </div>
</template>


<script>  
import categoriesQuery from '~/apollo/queries/category/categories'
import userQuery from '~/apollo/queries/user/user'
import gql from 'graphql-tag'
import { mapMutations } from 'vuex'

export default {  
    layout: 'withCategories',
    data() {
        return {
            category: Object,
            title: '',
            description: '',
            category: '',
            expirationDate: ''           
        }
    },
    
    apollo: {
        categories: {
            prefetch: true,
            query: categoriesQuery,
        },
        user: {
            prefetch: true,
            query: userQuery,
            variables () {
                return { id: this.$route.params.id }
            }
        },
    },
    methods: {
        newOffer (event) {
        this.$apollo
            .mutate({
            mutation: gql`
                mutation (
                    $title: String
                    $description: String
                    $category: ID
                    $expirationDate: DateTime
                    $user:ID
                          
                ){
                    createOffer(input: {
                        data: {
                            title: $title
                            description: $description
                            category: $category
                            expirationDate: $expirationDate
                            user: $user
                            state: 0
                        }
                    }) {
                        offer {
                            title
                            description 
                            expirationDate 
                            state
                            category {
                                id
                                title
                                }    
                            user{
                                id
                            }                     
                        }
                    }
                }
            `,
            variables: {
                title: this.title,
                description: this.description,
                category: this.category,
                user: this.$route.params.id,
                expirationDate: this.expirationDate
                }
            })
            .then((data) => {
            this.$router.push({name: 'member-id-dashboard', params: {id:this.$route.params.id}}),
            event.target.reset()
            })
            .catch((e) => {
            this.errors = e.graphQLErrors
            })
        },
        ...mapMutations({
            logout: 'auth/logout'
        })
    },
    computed: {
        username() {
            return this.$store.getters['auth/username']
        },
        id() {
            return this.$store.getters['auth/id']
        }
    }
}
</script>