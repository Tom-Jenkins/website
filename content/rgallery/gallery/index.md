+++
# Gallery section using the Blank widget and Gallery element (shortcode).
widget = "blank"  # See https://sourcethemes.com/academic/docs/page-builder/
headless = true  # This file represents a page section.
active = true  # Activate this widget? true/false
weight = 1  # Order that this section will appear.

title = "Data Vis Gallery"
subtitle = "Images created using R"
+++

{{< gallery >}}

<br/>
<br/>
Simulation showing the passive movement of particles as they are released from sites across southwest England in August 2010. This animation was created from the outputs of ocean drift models using Python {{< icon name="python" pack="fab" >}}
{{< video src="pylag_anim_14days_Twitter.mp4" controls="yes" >}}
<sub><sup>This simulation used the FVCOM ocean model and the particle tracking algorithm developed and implemented by Jim Clark (Plymouth Marine Laboratory).</sup></sub>