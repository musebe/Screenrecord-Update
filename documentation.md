### Cloudinary Screen Recorder
## Background
A screen recorder is a software that turns screen output into a video. It’s usefully applicable in use cases such as making videos of screen sequences to log results for troubleshooting, narration during capture, and creating software tutorials. 

## Prerequisites
Basic understanding of Javascript and React

## Setting Up the Sample Project
Using your terminal, generate a Next.js by using the create-next-app CLI:

`npx create-next-app screenrecorder`

Go to your project directory :

`cd screenrecorder`

## Cloudinary Setup

We begin with setting up a free Cloudinary account here. In your account’s dashboard, you will receive environment keys to use in integrating Cloudinary to your project.

Create a file named ‘.env.local’ in your project root directory and paste in the following:

```
".env.local"

CLOUDINARY_NAME=

CLOUDINARY_API_KEY = 

CLOUDINARY_API_SECRET=
```

Complete the above with information from your Cloudinary account.

Go to the ‘pages/api’ directory and create a folder called ‘utils’. Inside the folder create a file named ‘cloudinary’. And paste the following code.


 ```
 "pages/api/cloudinary"


require('dotenv').config();
const cloudinary = require('cloudinary');
cloudinary.config({
    cloud_name: process.env.CLOUDINARY_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_API_SECRET,
    upload_preset: process.env.CLOUDINARY_UPLOAD_PRESET,
});

module.exports = { cloudinary };
```

Our last Cloudinary setup phase will involve setting up our project’s backend. Inside the api directory, create a file named upload.js. We’ll use this component to upload the recorded files. The backend will receive files in base64 string format. The files will be uploaded to the user's web profile and the file's Cloudinary link sent back as a response.


```
"pages/api/cloudinary.js"


var cloudinary = require("cloudinary").v2;

cloudinary.config({
    cloud_name: process.env.CLOUDINARY_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_API_SECRET,
    upload_preset: process.env.CLOUDINARY_UPLOAD_PRESET,
});
export const config = {
    api: {
        bodyParser: {
            sizeLimit: "20mb",
        },
    },
};

export default async function handler(req, res) {
    let uploaded_url = ""
    const fileStr = req.body.data;

    if (req.method === "POST") {
        try {
            const uploadedResponse = await cloudinary.uploader.upload_large(
                fileStr,
                {
                    resource_type: "video",
                    chunk_size: 6000000,
                }
            );
            console.log("uploaded_url", uploadedResponse.secure_url)
            uploaded_url = uploadedResponse.secure_url
        } catch (error) {
            res.status(500).json({ error: "Something wrong" });
        }
        res.status(200).json({ data : uploaded_url });
    }
}
```

With our backend complete, let us create our screen recorder.
Create a folder named ‘components’ in your project root directory and inside it create a file named ‘record.js’. 

## Recording video output
Start by pasting the following code in your home component return statement. You can locate css files in the GitHub repo.
```
"pages/index.js"

    return (
      <div>
        <h1 className='elegantshadow'>Screen  Recorder</h1>
        <video ref={videoRef} className="video" width="600px" src={ link && link } controls></video><br/>
        <button onClick={recordHandler} className="record-btn">record</button>
      </div>
    )
```
With the css codes integrated, your UI will look like the below:

![complete UI](https://res.cloudinary.com/dogjmmett/image/upload/v1662905850/Screenshot_2022-09-11_at_17.16.27_o66hij.png "complete UI")

Now let's code the recorder. Notice the onClick function in our button. Let's start by creating the function. Inside it includes the stream of our display
 
 ```
 "pages/index"

 const recordHandler = async () => {
    let stream = await navigator.mediaDevices.getDisplayMedia({
                video: true
            });
    }
```
At this point, when you click the button, you should see the following popup

![stream popup](https://res.cloudinary.com/dogjmmett/image/upload/v1662907019/Screenshot_2022-09-11_at_17.18.55_vtuxkk.png "stream popup")

The following code will then use a media recorder to record the video.

 ```
 "pages/index"

 const recordHandler = async () => {
    let stream = await navigator.mediaDevices.getDisplayMedia({
                video: true
            });
    }
      //needed for better browser support
  const mime = MediaRecorder.isTypeSupported("video/webm; codecs=vp9") 
             ? "video/webm; codecs=vp9" 
             : "video/webm"
    let mediaRecorder = new MediaRecorder(stream, {
        mimeType: mime
    })

    //we have to start the recorder manually
    mediaRecorder.start()
```

while the screen records, the media recorder will provide data that can be stored in chunks. We shall store this data in a variable named `chunks`. When we stop recording the chunks will be converted to blob and sent to another function named `uploadHandler`. Since we want our recorded video to be played on the video tab, the `uploadHandler` function will convert the blob received to base64 using a file reader and upload the file to Cloudinary. The video's Cloudinary link will be received as a response and used as the src of the video element. We use state hooks to track the response link and useRef hook to reference the video element.

```
"pages/index.js"

const recordHandler = async () => {
        let stream = await navigator.mediaDevices.getDisplayMedia({
            video: true
        });
        const mime = MediaRecorder.isTypeSupported("video/webm; codecs=vp9") 
             ? "video/webm; codecs=vp9" 
             : "video/webm"
    let mediaRecorder = new MediaRecorder(stream, {
        mimeType: mime
    })

    let chunks = []
    mediaRecorder.ondataavailable = (e) => chunks.push(e.data);
    mediaRecorder.onstop = (e) => uploadHandler(new Blob(chunks, { type: 'video/webm' }));
   

    //we have to start the recorder manually
    mediaRecorder.start();
    }

    function readFile(file) {
        console.log("readFile()=>", file);
        return new Promise(function (resolve, reject) {
          let fr = new FileReader();
    
          fr.onload = function () {
            resolve(fr.result);
          };
    
          fr.onerror = function () {
            reject(fr);
          };
    
          fr.readAsDataURL(file);
        });
      }
    
      const uploadHandler = async(blob) => {
        let video = videoRef.current;
        // console.log(video);
        await readFile(blob).then((encoded_file) => {
            try {
              fetch('/api/cloudinary', {
                method: 'POST',
                body: JSON.stringify({ data: encoded_file }),
                headers: { 'Content-Type': 'application/json' },
              })
                .then((response) => response.json())
                .then((data) => {
                  setLink(data.data);
                });
            } catch (error) {
              console.error(error);
            }
          });
      }
```
Just like that, you have successfully created a recording app. Kindly go through the article to enjoy the experience.
Happy coding