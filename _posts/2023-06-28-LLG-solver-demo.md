---
title: Numerically solving the Landau–Lifshitz–Gilbert equation. 
image: /assets/20230628-imgs/llg-solver-screenshot.png
---  

The Landau–Lifshitz–Gilbert (LLG) equation is ubiquitous in the field of magnetism. It describes how a magnetic moment precesses around an external magnetic field, and under the influence of damping, gradually orientes itself alongside the external magnetic field. 

These concepts can sometimes be challenging to visualise, especially for those unfamiliar with the field. So a while back I wrote a script to solve the LLG equation and then as I got familiar with Streamlit, it seemed like a great project to convert into a web app. 

The app takes a single magnetic moment, which can be thought of as an arrow representing the direction of the magnetism in the block of magnetic material. It also takes an external magnetic field, which is another arrow along which the magnetic moment will want to align itself. You can then play around with the orientation of these relative to each other and also the size of the arrows. 

Run the simulation and the magnetic moment will precess around the external magnetic field, gradually falling in on it. The larger the damping the fast it will lie along the magnetic moment will align itself with the external magnetic field. 

This illustrates the role that magentic damping plays when reversing the magnetisation direction in a magnet. The higher the damping, the quicker it aligns along the new direction. Since switch nanomagnetic in hard disk drives is critical to storing and writing data, many researchers are trying to better understand how damping can be used to make this process as fast and as efficient as possible. 

The app is hosted on the Streamlit community cloud and I've simply embedded it below as an iframe in this webpage, but you can view it directly at its hosting URL [here](https://llgdemo.streamlit.app/).

The python code for the app is stored in my github repo [here](https://github.com/rich970/LLG-app). 


<iframe src="https://llgdemo.streamlit.app//?embedded=true"
		frameborder="2"
		marginheight="1000"
		marginwidth="500"
		width="100%"
		height="1080"
		scrolling="no">
</iframe>



