<template>
    <transition name="fade">

        <div @click="showInfo"
             ref="card"
             :loading="loading"
             class="card relative flex flex-wrap justify-center border border-lg border-50 overflow-hidden px-0 py-0 cursor-pointer"
        >
            <template v-if="loading">
                <div class="rounded-lg flex items-center justify-center absolute pin z-50">
                    <loader class="text-60" />
                </div>
            </template>

            <div v-if="file.mime != 'image'" v-html="file.thumb" class="mime-icon flex items-center justify-center  h-5/6">

            </div>

            <div  v-if="file.mime == 'image'" ref="image" class="image-block block w-full h-5/6">

            </div>

            <div class="missing p-8" v-if="missing">
                <p class="text-center leading-normal">
                    <a :href="file.name" class="text-primary dim" target="_blank">{{__('This image')}}</a> {{__('could not be found.')}}
                </p>
            </div>

            <div class="h-1/6 w-full text-center text-xs  border-t border-30 bg-50 flex items-center justify-center">
                {{ file.name | truncate(30) }}
            </div>

        </div>
    </transition>
</template>

<script>
import { Minimum } from 'laravel-nova';

export default {
    components: {
        //
    },

    props: {
        file: {
            type: Object,
            default: function() {
                return { name: '' };
            },
            required: true,
        },
    },

    data: () => ({
        loading: true,
        missing: false,
    }),

    mounted() {
        if (this.file.mime == 'image') {
            Minimum(
                window.axios.get(this.file.thumb, {
                    responseType: 'blob',
                })
            )
                .then(({ headers, data }) => {
                    const blob = new Blob([data], { type: headers['content-type'] });
                    let imageDiv = document.createElement('div');
                    let imageBlog = null;

                    imageBlog = window.URL.createObjectURL(blob);
                    imageDiv.style.backgroundImage = "url('" + imageBlog + "')";
                    imageDiv.className = 'block w-full h-full bg-center bg-cover h-2/3';
                    imageDiv.draggable = false;
                    this.$refs.image.appendChild(imageDiv);
                    this.loading = false;
                })
                .catch(error => {
                    if (error) {
                        //defaulImage
                        let imageDiv = document.createElement('div');
                        imageDiv.style.backgroundImage = "url('" + this.file.thumb + "')";
                        imageDiv.className = 'block w-full h-full bg-center bg-cover h-2/3';
                        imageDiv.draggable = false;
                        this.$refs.image.appendChild(imageDiv);
                        this.loading = false;
                    }
                });
        } else {
            this.loading = false;
        }
    },
    methods: {
        showInfo() {
            this.$emit('showInfo', this.file);
        },
    },
    filters: {
        truncate: function(text, stop, clamp) {
            return text.slice(0, stop) + (stop < text.length ? clamp || '...' : '');
        },
    },
    watch: {
        'file.loading': function(value) {
            if (value == true) {
                this.loading = true;
            } else {
                this.loading = false;
            }
        },
    },
};
</script>

<style scoped  lang="scss">
.card {
    padding: 0 !important;

    &:hover {
        > .image-block,
        > .mime-icon {
            opacity: 0.5;
        }
    }
}

.h-5\/6 {
    height: 83.33333%;
}

.h-1\/6 {
    height: 16.66667%;
}
</style>

<style>
.svg-mime {
    width: 80px;
    height: 75%;
    fill: #62676d;
}
</style>
