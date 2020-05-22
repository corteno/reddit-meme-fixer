<template>
    <div class="home-wrapper">
        <h1 class="page-title">Reddit Title Fixer</h1>
        <h2 class="page-subtitle">
            Don't you hate when the punchline of a meme is in the title and can't link it to your friends?
            Fear no more, just paste the link and you'll get your meme back!
        </h2>
        <form @submit="onSubmit" class="form-wrapper">
            <input type="text" class="text-input" v-model="inputValue">
            <input type="submit" value="Submit" class="submit-button">
        </form>
        <p class="error-message">{{this.errorMessage}}</p>

        <Canvas ref="canvas" width="600" height="600"></Canvas>

        <img src="" alt="" ref="img" style="">

    </div>
</template>

<script>
    import psl from 'psl';
    import axios from 'axios';

    export default {
        name: "Home",
        computed: {
            inputValue: {
                get() {
                    return this.value;
                },
                set(val) {
                    this.value = val;
                }
            }
        },
        data: function () {
            return {
                value: 'https://www.reddit.com/r/modernwarfare/comments/go38jw/the_entire_playerbase_once_price_drops_in_season_4/',
                errorMessage: '',
                textBlockHeight: 75,
                characterLimitPerLine: '55'
            }

        },
        methods: {
            onSubmit: function (e) {
                e.preventDefault();
                if (this.validateUrl(this.value)) {
                    let url = this.value + ".json";

                    axios.post('http://192.168.0.87:3000/meme', {
                        data: url
                    }).then((response) => {
                        //console.log(response.data);
                        this.displayOnCanvas(response.data);
                        //this.$refs.img.src = response.data.img;
                    });
                }
            },
            displayOnCanvas(data) {
                let canvas = this.$refs.canvas;
                let context = canvas.getContext('2d');
                let imageObj = new Image();
                imageObj.src = data.img;

                imageObj.onload = () => {
                    //let ratio = imageObj.width / imageObj.height;




                    context.canvas.width = imageObj.width;
                    context.canvas.height = imageObj.height + this.textBlockHeight;

                    context.drawImage(imageObj, 0, this.textBlockHeight);

                    context.fillRect(0, 0, imageObj.width, this.textBlockHeight);


                    context.font = "26px Arial"
                    context.fillStyle = 'white';
                    context.textAlign = 'center';

                    if (data.title.length > this.characterLimitPerLine) {
                        //console.log(this.getSpaces(data.title));
                        let newTitle = this.insertLineBreak(data.title, this.getSpaces(data.title));

                        for (let i = 0; i < newTitle.length; i++){
                            context.fillText(newTitle[i], context.canvas.width / 2, 30 * (i + 1));
                        }

                        newTitle.forEach((string) => {
                            console.log(string);
                        });

                    } else {

                        context.fillText(data.title, context.canvas.width / 2, 45);
                    }



                };

                imageObj.src = data.img;


            },
            getSpaces(string) {
                let spaces = [];

                for (let i = 0; i < string.length; i++){
                    //console.log(string[i]);
                    if(string[i] === ' '){
                        spaces.push(i);
                    }
                }
                return spaces;
            },
            insertLineBreak(string, spaces){
                let closest = spaces.reduce((prev, curr) => {
                    return (Math.abs(curr - this.characterLimitPerLine) < Math.abs(prev - this.characterLimitPerLine) ? curr : prev);
                });



                //console.log("closest: " + closest);
                return [string.slice(0, closest + 1), string.slice(closest + 1, string.length-1)];
             },
            setErrorMessage(message) {
                this.errorMessage = message;
                setTimeout(() => {
                    this.errorMessage = '';
                }, 5000);
            },
            validateUrl(str) {
                let pattern = new RegExp('^(https?:\\/\\/)?' + // protocol
                    '((([a-z\\d]([a-z\\d-]*[a-z\\d])*)\\.)+[a-z]{2,}|' + // domain name
                    '((\\d{1,3}\\.){3}\\d{1,3}))' + // OR ip (v4) address
                    '(\\:\\d+)?(\\/[-a-z\\d%_.~+]*)*' + // port and path
                    '(\\?[;&a-z\\d%_.~+=-]*)?' + // query string
                    '(\\#[-a-z\\d_]*)?$', 'i'); // fragment locator

                let isUrl = !!pattern.test(str);

                if (isUrl) {
                    let domainName = psl.get(this.extractHostName(str));
                    if (domainName === 'reddit.com') {
                        return true;
                    }
                }
                this.setErrorMessage('Invalid URL, can only fetch memes from reddit.com');
                return false
            },
            extractHostName(url) {
                let hostname;
                //find & remove protocol (http, ftp, etc.) and get hostname

                if (url.indexOf("//") > -1) {
                    hostname = url.split('/')[2];
                } else {
                    hostname = url.split('/')[0];
                }

                //find & remove port number
                hostname = hostname.split(':')[0];
                //find & remove "?"
                hostname = hostname.split('?')[0];

                return hostname;
            }

        }
    }
</script>

<style scoped>

    .form-wrapper {
        height: 30px;
    }

    .text-input {
        height: 100%;
        max-width: 700px;
        width: 100%;
        font-size: 16px;
        padding: 0 5px;
    }

    .submit-button {
        appearance: none;
        -webkit-appearance: none;
        -moz-appearance: none;
        height: 100%;
    }
</style>