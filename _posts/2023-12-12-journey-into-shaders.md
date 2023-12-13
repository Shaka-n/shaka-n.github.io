---
title: "Journey Into Shaders, Sea Bob"
tags: glsl, shaders, graphics
---

I've been messing around with writing shaders recently. I'm just learning for fun. It appeals to me because of how fundamental a practice it is and one that hasn't changed all that dramatically (depending on who you ask) since the introduction of the dedicated GPU. Being the cynic and luddite that I am (joke), I'm even doubtful that this will change with the broader adoption of Gaussian Splatting and the Vulkan API (I don't think these are necessary for making good video games, but that's another blog post). All that aside, graphics programming: it's what's for dinner.

To keep things simple and enjoyable enough for me to pursue this alongside every other side project in my life, I'm starting with GLSL, Shadertoy.com, Iñigo Quiles' website, and The Book of Shaders. I've slightly cooled on Shadertoy due to its odd choice of naming its uniforms different from most other GLSL implementations, making it a chore to port my local code into the site. Also, probably because I use Brave, I can't get the in-browser editor to save my dang code, so screw 'em babe. As I publish new shaders on my journey, I'll post them here. As I come to shaping algorithms, I'll add them to [my repo where I collect the shaping functions](https://github.com/Shaka-n/shader-shaping-functions) I've spent an appreciable amount of time trying to understand. This is all very painterly to me; lots of magic numbers, intuitions, and vibes. I don't recommend trying to learn much of anything here.

I use [Patricio Gonzalez's glslCanvas](https://github.com/patriciogonzalezvivo/glslCanvas) to render my shaders on this site, and [glslViewer](https://github.com/patriciogonzalezvivo/glslViewer) by the same author to develop them locally.

So here's my first two shaders. The first one I call *Sea Bob*, and is meant to invoke the feeling of closing your eyes on a sunny day, bobbing just above the surface of the ocean as the waves gently lap over your head. Artsy, eh? I made this for one of the exercises in *The Book of Shaders*.

<canvas class="glslCanvas" data-fragment="
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform float u_time;

float doubleCubicSeatWithLinearBlend (float x, float a, float b){

  float epsilon = 0.00001;
  float min_param_a = 0.0 + epsilon;
  float max_param_a = 1.0 - epsilon;
  float min_param_b = 0.0;
  float max_param_b = 1.0;
  a = min(max_param_a, max(min_param_a, a));  
  b = min(max_param_b, max(min_param_b, b)); 
  b = 1.0 - b; //reverse for intelligibility.
  
  float y = 0.0;
  if (x <= a){
    y = b*x + (1.0-b)*a*(1.0-pow(1.0-x/a, 3.0));
  } else {
    y = b*x + (1.0-b)*(a + (1.0-a)*pow((x-a)/(1.0-a), 3.0));
  }
  return y;
}


vec3 colorA = vec3(0.053,0.670,0.316);
vec3 colorB = vec3(0.079,0.985,0.976);

void main() {
    vec3 color = vec3(0.0);
    
    float dbc = doubleCubicSeatWithLinearBlend(sin(u_time), 0.324, 0.5);
	float pct = sin(dbc);
    // float pct = 1.0 - pow(max(0.0, abs(sin(dbc))), 0.208);

    // Mix uses pct (a value from 0-1) to
    // mix the two colors
    color = mix(colorA, colorB, pct);

    gl_FragColor = vec4(color,1.0);
}" width="500" height="500"></canvas>

{% highlight glsl%}
// Sea Bob
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform float u_time;

float doubleCubicSeatWithLinearBlend (float x, float a, float b){

  float epsilon = 0.00001;
  float min_param_a = 0.0 + epsilon;
  float max_param_a = 1.0 - epsilon;
  float min_param_b = 0.0;
  float max_param_b = 1.0;
  a = min(max_param_a, max(min_param_a, a));  
  b = min(max_param_b, max(min_param_b, b)); 
  b = 1.0 - b; //reverse for intelligibility.
  
  float y = 0.0;
  if (x <= a){
    y = b*x + (1.0-b)*a*(1.0-pow(1.0-x/a, 3.0));
  } else {
    y = b*x + (1.0-b)*(a + (1.0-a)*pow((x-a)/(1.0-a), 3.0));
  }
  return y;
}


vec3 colorA = vec3(0.053,0.670,0.316);
vec3 colorB = vec3(0.079,0.985,0.976);

void main() {
    vec3 color = vec3(0.0);
    
    float dbc = doubleCubicSeatWithLinearBlend(sin(u_time), 0.324, 0.5);
	float pct = sin(dbc);
    // float pct = 1.0 - pow(max(0.0, abs(sin(dbc))), 0.208);

    // Mix uses pct (a value from 0-1) to
    // mix the two colors
    color = mix(colorA, colorB, pct);

    gl_FragColor = vec4(color,1.0);
}
{% endhighlight %}

...and this one's called *Circles*. Technically, it's the first thing I made on my own. 

<iframe width="640" height="360" frameborder="0" src="https://www.shadertoy.com/embed/csKBRy?gui=true&t=10&paused=true&muted=false" allowfullscreen></iframe>

{% highlight glsl %}
// Circles
void disk(vec2 uv, vec2 center, float radius, vec3 color, inout vec3 pixel) {
	if( length(uv-center) < radius) {
		pixel = color;
	}
}

void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
// what is happening
    // Normalized pixel coordinates (from 0 to 1)
    vec2 uv = vec2(fragCoord * 2.0 - iResolution.xy) / iResolution.y;

    vec3 bgCol = vec3(0.3);
	vec3 colBlue = vec3(0.216, 0.471, 0.698);
	vec3 colRed = vec3(1.00, 0.329, 0.298);
	vec3 colYellow = vec3(0.867, 0.910, 0.247);

    vec3 pixel = bgCol;
 
    disk(uv, vec2(0.4, 0.1), 0.2, colBlue, pixel);
    disk(uv, vec2(0.8, -0.5), 0.6, colRed, pixel);
    disk(uv, vec2(-0.2, 0.6), 0.4, colYellow, pixel);

    // Output to screen
    fragColor = vec4(pixel, 1.0);

}
{% endhighlight %}

That's all for now! Be back soon with more flashing colors and abstract shapes.