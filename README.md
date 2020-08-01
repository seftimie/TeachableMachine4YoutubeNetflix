# TeachableMachine4YoutubeNetflix
I spent one quality hour with [my kido (sir Karl)](https://www.youtube.com/channel/UCHV6pF1KjgCJB4M__4ZYISg) training a ML model to play/pause videos from distance on Youtube, Netflix or HBO in a hacky way. We've been using [Teachable Machine (from Google AI Experiments)](https://teachablemachine.withgoogle.com/), some Javascript and Chrome Developer Tools. The kid enjoyed a lot the output.

![Neo, Matrix stopping videos](neo.jpg?raw=true "Neo, from Matrix stopping videos on Youtube/Netflix/Hbo/Vimeo etc")

[Full demo (video) of TeachableMachine4YoutubeNetflix](https://www.youtube.com/watch?v=iYZbfsHiqXk)



### Solution (step by step):

* I introduced sir K to Teachable Machine creating an image model with two classes: "Play" and "Nothing". On "Play" we recorded 150 images with our hands rising or showing 🤚 to the webcam. On "Nothing" we recorded 150 images with us, our home, dog, cat, food with no hands rising or showing 🤚. 

* We've done some testing on Teachable Machine's playground to validate that our model was working;

* Export & use the Model: We chose to upload our model to Google Cloud Platform and get a nice shareable link; Then we grabbed the Teachable Machine's JS code and tweaked it a little.

* With Chrome, goto: https://www.youtube.com/watch?v=hFZFjoX2cGg

* Open Chrome Developer Tools -> Console and Inject the JS code (copy + paste) in this order:

```javascript
var script = document.createElement('script');
script.src = 'https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js';
script.type = 'text/javascript';
document.getElementsByTagName('head')[0].appendChild(script);
```

```javascript
var script = document.createElement('script');
script.src = 'https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js';
script.type = 'text/javascript';
document.getElementsByTagName('head')[0].appendChild(script);
```

```javascript
const URL = "https://teachablemachine.withgoogle.com/models/z7ROgIrpA/";
let model, webcam, labelContainer, maxPredictions;

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

const requestTimeoutNoRaf = (fn, delay, registerCancel) => {
  const timeoutId = setTimeout(fn, delay);
  registerCancel(() => clearTimeout(timeoutId));
}

async function init()
{
    const modelURL = URL + "model.json";
    const metadataURL = URL + "metadata.json";

    model = await tmImage.load(modelURL, metadataURL);
    maxPredictions = model.getTotalClasses();

    const flip = true;
    webcam = new tmImage.Webcam(200, 200, flip);
    await webcam.setup();
    await webcam.play();
    window.requestAnimationFrame(loop);
}

async function loop()
{
    webcam.update();
    await predict();
    window.requestAnimationFrame(loop);
}

async function predict()
{
    const prediction = await model.predict(webcam.canvas);
    for (let i = 0; i < maxPredictions; i++)
    {
        const classPrediction = prediction[i].className + ": " + prediction[i].probability.toFixed(2);
        console.log("TF detected: "+classPrediction);
        var url = document.domain;
        if (prediction[i].className=="Play" && prediction[i].probability.toFixed(2)=="1.00")
        {
                console.log("Play or Stop video");
                if (url.includes("youtube"))
                {
                    console.log("on youtube ");
                    var e = new KeyboardEvent('keydown',{'keyCode':32,'which':32});
                    document.dispatchEvent(e);    
                }

                if (url.includes("netflix"))
                {
                    console.log("on netflix ");
                    if (document.getElementsByClassName("touchable PlayerControls--control-element nfp-button-control default-control-button button-nfplayerPlay").length)
                    {
                        console.log("press pause");
                        document.getElementsByClassName("touchable PlayerControls--control-element nfp-button-control default-control-button button-nfplayerPause")[0].click();
                    }
                    else
                    {
                        console.log("press play");
                        document.getElementsByClassName("touchable PlayerControls--control-element nfp-button-control default-control-button button-nfplayerPlay")[0].click();
                    }
                }
                await sleep(1000);
        }
    }
}

init();
```

* At this point Chrome will ask you for permissions to use your webcam. Just allow it!


* When the webcam will see  a hand rising will stop or play the video (aka push spacebar)


### Todos:

* Not the best code out there... but hey it's just a nice demo;
* maybe someday, someone (not me) could write an Chrome Extension where you can specify your custom model url and some gestures (eg: when class "x" probability 1.00 then "press space");


Enjoy it !


PD: Don't use this for gym sessions 💪💪💪 - it's a disaster !!!! [Karl's mommy](https://www.linkedin.com/in/cornelia-nicoleta-radulescu-6b3b7b16a/) just tried it out and was a big fail 😐
