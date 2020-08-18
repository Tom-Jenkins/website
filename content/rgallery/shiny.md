+++
# Shiny widget
widget = "blank"
headless = true
active = true
weight = 2

# Slide height (optional).
# E.g. `500px` for 500 pixels or `calc(100vh - 70px)` for full screen.
height = ""

[design]
  # Choose how many columns the section has. Valid values: 1 or 2.
  columns = "1"

[design.spacing]
  # Customize the section spacing. Order is top, right, bottom, left.
  padding = ["20px", "0", "20px", "0"]
  
+++

<iframe height="600" width="100%" frameborder="yes" src="https://tomjenkins.shinyapps.io/particle_drift_app/"> </iframe>

<br/>
<br/>

Simulation showing the passive movement of particles as they are released from sites across southwest England in August 2010. This animation was created from the outputs of ocean drift models using Python {{< icon name="python" pack="fab" >}}
{{< video src="pylag_anim_14days_Twitter.mp4" controls="yes" >}}
<sub><sup>This simulation used the FVCOM ocean model and the particle tracking algorithm developed and implemented by Jim Clark (Plymouth Marine Laboratory).</sup></sub>
