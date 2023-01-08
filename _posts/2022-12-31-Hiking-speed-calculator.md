---
title: An simple app built using streamlit to calculate hiking times for walks. 
image: 
---  

I've been keen to try out Streamlit, a python package for rapidly developing web applications. The focus of the package is for deploying Machine Learning models, and I have to say, it makes really light work of developin simple web applications like this one. They provide well automated tools which make it a breeze to add user controls such as sliders and buttons.  

I've been planning some multi day hikes and been interested in estimating the maximum distance I could cover in a day based on my usual pace for day hikes and accounting for additional rest times. So I built a simple calculator for a bit of fun and a chance to try out the Streamlit module. 

The app that I've made it hosted on the Streamlit community cloud and I've simply embeded it below as an iframe in this webpage.


<iframe src="https://rich970-hiking-trip-app-hiking-trip-app-0f5ced.streamlit.app/?embedded=true"
		frameborder="2"
		marginheight="1000"
		marginwidth="500"
		width="100%"
		height="1080"
		scrolling="no">
</iframe>

The python code for the app is stored in my github repo [here](https://github.com/rich970/hiking-trip-app) which is hooked up to the Streamlit community cloud. This process is as simple as creating an account and pasting in a link to the github repo. The instructions can be found [here](https://docs.streamlit.io/streamlit-cloud/get-started/deploy-an-app) and Streamlit takes care of the rest. Just note, that you may need to add a requirements.txt file to ensure additional packages are installed, as I needed to do to include matplotlib installation.
