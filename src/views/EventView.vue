<template>
    <section v-if="event">
        <article>
            <h1>
            {{ event.name }}
            </h1>
            <img v-if="event.images" :src="event.images[0].url">
            <img v-if="event.seatmap" :src="event.seatmap.staticUrl">
            <p v-if="!event.dates.start.dateTBD && !event.dates.start.dateTBA">Fecha del evento: {{ event.dates.start.localDate }}</p>
            <p v-if="event._embedded.venues">En: {{ event._embedded.venues[0].country.name }}, {{ event._embedded.venues[0].city.name }}</p>
            <p v-if="event.info">{{ event.info }}</p>
            <button v-if="event.seatmap">Ver mapa del recinto</button>
            <a v-if="event.url" :href="event.url" target="_blank">Comprar entradas</a>
        </article>
        <article v-if="user && !user.message">
        <form @submit.prevent="sendComment">
            <textarea v-model="commentBox" id="commentBox"></textarea>
            <button @submit="sendComment">Enviar comentario</button>
        </form>
        </article>
        <article v-if="comments">
            <p v-for="comment in comments">{{ comment.userId }} {{ comment.commentText }}</p>
        </article>
    </section>  
</template>

<script>
    import {useCookies} from '@vueuse/integrations/useCookies'
    const cookies = useCookies(["token"])
    export default {
        data() {
            return {
                event: null,
                comments: null,
                user: null,
                commentBox: ""
            }
        },
        props: {
            id: {type: String, required: true}
        },
        async created() {
            const eventData = await fetch(`https://app.ticketmaster.com/discovery/v2/events/${this.id}.json?apikey=S1sDAS05dZI5JmtvdarQaZN5tFxkOUpr`)
            const eventResponse = await eventData.json()
            this.event = eventResponse
        },
        async mounted() {
            //Si se elimina la cookie desde el navegador y se envia el comentario, todavia dejara enviarlo con las credenciales anteriores
            const userData = await fetch("http://localhost:8000/api/user", {
            method: 'get',
            headers: {
                'Authorization': 'Bearer ' + cookies.get("token")
            }})
            const userResponse = await userData.json()
            if (userResponse.message) {
                cookies.remove("token", {path:"/"})
            }
            else {
                this.user = userResponse
            }
            this.updateComments()
        },
        methods: {
            async sendComment() {
                console.log(this.user.id, this.$props.id, this.commentBox)
                const commentBoxData = await fetch('http://localhost:8000/api/comments', {
                    method: "post",
                    headers: {
                        'Accept': 'application/json',
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        userId: this.user.id,
                        eventId: this.$props.id,
                        commentText: this.commentBox,
                    })
                })
                const commentBoxResponse = await commentBoxData.json()
                this.updateComments()
            },
            async updateComments() {
                const commentData = await fetch(`http://localhost:8000/api/comments/${this.$props.id}`)
                const commentResponse = await commentData.json()
                this.comments = commentResponse
            }
        }
    }
</script>