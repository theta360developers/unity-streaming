This documentation was [originally written](http://tips.hecomi.com/entry/2015/10/11/211456) by
[hecomi](http://tips.hecomi.com/)  in Japanese. It was translated into English
by the THETA Unofficial Guide team.

To view a THETA live streaming video using Unity, I made a an appropriately UV developed sphere and a shader
appropriate to AlphaBlend border. Ultimately, for the purpose of making a full sphere with the THETA S,
it would be much higher quality and more convenient (can use Skybox too!) to use the fragment shader,
made by Nora ([@Stereoarts](https://twitter.com/stereoarts), which directly writes Equirectangular onto a plane.

[Stereoarts Homepage](http://stereoarts.jp/)

[@Stereoarts tweet](https://twitter.com/Stereoarts/status/647737666520248321?ref_src=twsrc%5Etfw): I’ve released a Theta Shader Pack. A shader for converting THETA / THETA S full sphere video to Enquirectangular in Unity and supporting scripts. [GitHub](https://github.com/theta360developers/unity-streaming/tree/master/stereoarts)

[original zipped archive](stereoarts.jp/ThetaShaderPack_20150926.zip )

For this article, I wanted to jot down my techniques as well.

Sample

[GitHub](https://github.com/theta360developers/unity-streaming/tree/master/goroman)
[Original](https://dl.dropboxusercontent.com/u/7131835/Programs/ThetaS_LiveView_Sample.unitypackage)

Example of taking a video with THETA

When taking a video with m15, the circles overlapped. The THETA S gives beautifully separated spheres. The angle covered in one sphere is slightly larger than 180 degrees.

![dual fish eye image](http://theta360.guide/blog/img/2015-12/dual-fish-eye.jpg)

![dual fish eye image with tripod](http://theta360.guide/blog/img/2015-12/dual-fisheye-tripod.jpg)

In doing this, I've made an adjustment using sample texture, which Goroman filmed using WebCamTexture.

Making a sphere with good UV setting

Working with Maya LT, the UV comes out like this, if you make a normal sphere.

![Maya LT UV Mapping](http://theta360.guide/blog/img/2015-12/MayaLT-UV-mapping.png)

It would look like below, if you make a plane with the UV.

![Maya LT UV Mapping 2](http://theta360.guide/blog/img/2015-12/MayaLT-UV-mapping-2.png)

All it needs is to be cut in half and be moved appropriately.

![Maya LT UV Mapping 3](http://theta360.guide/blog/img/2015-12/MayaLT-UV-mapping-3.png)

![Maya LT UV Mapping 4](http://theta360.guide/blog/img/2015-12/MayaLT-UV-mapping-4.png)

It looks like this. (I did not adjust it, so it might be slightly off.)

![Crescent Moon image](http://theta360.guide/blog/img/2015-12/crescent-moon.png)

Actually, I wanted to use alphablend for the border, so I used 2 overlapping half spheres instead of one sphere. The UV border is adequately stretched manually.

![Overlapping](http://theta360.guide/blog/img/2015-12/overlapping.png)

Incidentally, surface is set to face inward, by reversing all normal vectors. UV position and size are fine to adjust later with shader.


Setting with Unity

Import the Maya LT built model with Unity, and put the camera in the center. Write a shader, so the model's UV position can be adjusted or can alphablend. In order to control the drawing order and to prevent the border from changing at certain orientations, each half sphere has a different shader.

	Shader "Theta/Sphere1" {
	    Properties {
	        _MainTex ("Base (RGB)", 2D) = "white" {}
	        _AlphaBlendTex ("Alpha Blend (RGBA)", 2D) = "white" {}
	        _OffsetU ("Offset U", Range(-0.5, 0.5)) = 0
	        _OffsetV ("Offset V", Range(-0.5, 0.5)) = 0
	        _ScaleU ("Scale U", Range(0.8, 1.2)) = 1
	        _ScaleV ("Scale V", Range(0.8, 1.2)) = 1
	        _ScaleCenterU ("Scale Center U", Range(0.0, 1.0)) = 0
	        _ScaleCenterV ("Scale Center V", Range(0.0, 1.0)) = 0
	    }
	    SubShader {
	        Tags { "RenderType" = "Transparent" "Queue" = "Background" }
	        Pass {
	            Name "BASE"

	            Blend SrcAlpha OneMinusSrcAlpha
	            Lighting Off
	            ZWrite Off

	            CGPROGRAM
	            #pragma vertex vert_img
	            #pragma fragment frag

	            #include "UnityCG.cginc"

	            uniform sampler2D _MainTex;
	            uniform sampler2D _AlphaBlendTex;
	            uniform float _OffsetU;
	            uniform float _OffsetV;
	            uniform float _ScaleU;
	            uniform float _ScaleV;
	            uniform float _ScaleCenterU;
	            uniform float _ScaleCenterV;

	            float4 frag(v2f_img i) : COLOR {
	                // 中心位置や大きさを微調整
	                float2 uvCenter = float2(_ScaleCenterU, _ScaleCenterV);
	                float2 uvOffset = float2(_OffsetU, _OffsetV);
	                float2 uvScale = float2(_ScaleU, _ScaleV);
	                float2 uv =  (i.uv - uvCenter) * uvScale + uvCenter + uvOffset;
	                // アルファブレンド用のテクスチャを参照してアルファを調整
	                float4 tex = tex2D(_MainTex, uv);
	                tex.a *= pow(1.0 - tex2D(_AlphaBlendTex, i.uv).a, 2);
	                return tex;
	            }
	            ENDCG
	        }
	    }
	}

Here's a second section of code.

		Shader "Theta/Sphere2" {
		    Properties {
		        _MainTex ("Base (RGB)", 2D) = "white" {}
		        _AlphaBlendTex ("Alpha Blend (RGBA)", 2D) = "white" {}
		        _OffsetU ("Offset U", Range(-0.5, 0.5)) = 0
		        _OffsetV ("Offset V", Range(-0.5, 0.5)) = 0
		        _ScaleU ("Scale U", Range(0.8, 1.2)) = 1
		        _ScaleV ("Scale V", Range(0.8, 1.2)) = 1
		        _ScaleCenterU ("Scale Center U", Range(0.0, 1.0)) = 0
		        _ScaleCenterV ("Scale Center V", Range(0.0, 1.0)) = 0
		    }
		    SubShader {
		        Tags { "RenderType" = "Transparent" "Queue" = "Background+1" }
		        UsePass "Theta/Sphere1/BASE"
		    }
		}


As below, for alphablend, have a texture made, that is alpha adjusted to UV. I made adjustment for perfectly fit, by exporting UV with postscript and reading with illustrator (white circle inside is alpha=1; around the circle, from inside to outside, changes from 1 to 0; outside will not be used so whatever fits.)

![Two Circles](http://theta360.guide/blog/img/2015-12/two-circles.png)

Then, adjust the parameters and you've got a whole sphere.

![Parameters](http://theta360.guide/blog/img/2015-12/parameters.png)

![Unity Sphere](http://theta360.guide/blog/img/2015-12/sphere-unity.png)

![Realtime Stitching](http://theta360.guide/blog/img/2015-12/realtime-stitching.png)

Changing into Equirectangular

I tried it with a modified vertex shader.

	Shader "Theta/Equirectangular1" {
	    Properties {
	        _MainTex ("Base (RGB)", 2D) = "white" {}
	        _AlphaBlendTex ("Alpha Blend (RGBA)", 2D) = "white" {}
	        _OffsetU ("Offset U", Range(-0.5, 0.5)) = 0
	        _OffsetV ("Offset V", Range(-0.5, 0.5)) = 0
	        _ScaleU ("Scale U", Range(0.8, 1.2)) = 1
	        _ScaleV ("Scale V", Range(0.8, 1.2)) = 1
	        _ScaleCenterU ("Scale Center U", Range(0.0, 1.0)) = 0
	        _ScaleCenterV ("Scale Center V", Range(0.0, 1.0)) = 0
	        _Aspect ("Aspect", Float) = 1.777777777
	    }
	    SubShader {
	        Tags { "RenderType" = "Transparent" "Queue" = "Background" }
	        Pass {
	            Name "BASE"

	            Blend SrcAlpha OneMinusSrcAlpha
	            Lighting Off
	            ZWrite Off

	            CGPROGRAM
	            #pragma vertex vert
	            #pragma fragment frag
	            #define PI 3.1415925358979

	            #include "UnityCG.cginc"

	            uniform sampler2D _MainTex;
	            uniform sampler2D _AlphaBlendTex;
	            uniform float _OffsetU;
	            uniform float _OffsetV;
	            uniform float _ScaleU;
	            uniform float _ScaleV;
	            uniform float _ScaleCenterU;
	            uniform float _ScaleCenterV;
	            uniform float _Aspect;

	            struct v2f {
	                float4 position : SV_POSITION;
	                float2 uv       : TEXCOORD0;
	            };

	            v2f vert(appdata_base v) {
	                float4 modelBase = mul(_Object2World, float4(0, 0, 0, 1));
	                float4 modelVert = mul(_Object2World, v.vertex);

	                float x = modelVert.x;
	                float y = modelVert.y;
	                float z = modelVert.z;

	                float r = sqrt(x*x + y*y + z*z);
	                x /= 2 * r;
	                y /= 2 * r;
	                z /= 2 * r;

	                float latitude  = atan2(0.5, -y);
	                float longitude = atan2(x, z);  

	                float ex = longitude / (2 * PI);
	                float ey = (latitude - PI / 2) / PI * 2;
	                float ez = 0;

	                ex *= _Aspect;

	                modelVert = float4(float3(ex, ey, ez) * 2 * r, 1);

	                v2f o;
	                o.position = mul(UNITY_MATRIX_VP, modelVert);
	                o.uv       = MultiplyUV(UNITY_MATRIX_TEXTURE0, v.texcoord);
	                return o;
	            }    

	            float4 frag(v2f i) : COLOR {
	                float2 uvCenter = float2(_ScaleCenterU, _ScaleCenterV);
	                float2 uvOffset = float2(_OffsetU, _OffsetV);
	                float2 uvScale = float2(_ScaleU, _ScaleV);
	                float2 uv =  (i.uv - uvCenter) * uvScale + uvCenter + uvOffset;
	                float4 tex = tex2D(_MainTex, uv);
	                tex.a *= pow(1.0 - tex2D(_AlphaBlendTex, i.uv).a, 2);
	                return tex;
	            }
	            ENDCG
	        }
	    }
	}

Here's a second section of code.

	Shader "Theta/Equirectangular2" {
	    Properties {
	        _MainTex ("Base (RGB)", 2D) = "white" {}
	        _AlphaBlendTex ("Alpha Blend (RGBA)", 2D) = "white" {}
	        _OffsetU ("Offset U", Range(-0.5, 0.5)) = 0
	        _OffsetV ("Offset V", Range(-0.5, 0.5)) = 0
	        _ScaleU ("Scale U", Range(0.8, 1.2)) = 1
	        _ScaleV ("Scale V", Range(0.8, 1.2)) = 1
	        _ScaleCenterU ("Scale Center U", Range(0.0, 1.0)) = 0
	        _ScaleCenterV ("Scale Center V", Range(0.0, 1.0)) = 0
	        _Aspect ("Aspect", Float) = 1.777777777
	    }
	    SubShader {
	        Tags { "RenderType" = "Transparent" "Queue" = "Background+1" }
	        UsePass "Theta/Equirectangular1/BASE"
	    }
	}

![Results](http://theta360.guide/blog/img/2015-12/results.png)

When taking a look at the mesh, it moves around like this.

![Results Mesh](http://theta360.guide/blog/img/2015-12/results-mesh.png)

Because polygon did not fit, there is a blank space in the corner. This could have been avoided if we have used a direct fragment shader like Nora.

Conclusion

It looks like there’s the possibility of multiple fun topics here like spherical AR and Stabilization. After the THETA S goes on sale, I would love to play with it more.

Update March 9, 2016

## Maya Planar UV Mapping Instruction

Thanks to [@hecomi](https://twitter.com/hecomi) for providing this video that shows how to tweak
the UV mapping. He produced it to help Flex van Geuns who was trying to use the
UV mapping with webgl.

{% include youtubePlayer.html id="FfhZltDqJ2U" %}

![Analytics](https://ga-beacon.appspot.com/UA-73311422-5/unity-streaming)
