# Project Title
Random Space Filling Tiling

## Authors
- Bram Stolk, (stolk)

## Acknowledgements
- Concept was pioneered by John Shier (http://www.john-art.com/)
- Concept was described by Paul Bourke (http://paulbourke.net/texture_colour/randomtile/)

## Description
Randomly tiling a bounded plane with an infinite number of non overlapping shapes is an interesting premise.
To avoid running out of space one has to shrink each additional shape.
But how much to shrink them? Shrink too fast, and the space will look empty.
Shrink them too slow, and you cannot find an empty spot for the next shape.
It turns out that the area size should follow a pow( i, -c ) sequence.

<I>i is the iteration number, and c is a tuned constant larger than 1 and lower than 1.5.</I>

## Link to Prototype
[Github project with source code](https://github.com/stolk/devart-template/tree/master/project_code "Source Code for this project.")

## Example Code
This piece of code shows how AVX allows you to consider eight edges w.r.t a point, in a single go.
Another nice property of the code is that there is no branching, no looping.
```
// 8-way SIMD calculation of winding number of a point versus a polygon size 8.
int winding_number_8( float px, float py, __m256 vx, __m256 vy, __m256 wx, __m256 wy )
{
        const __m256  zero8  = _mm256_setzero_ps();
        const __m256i zero8i = _mm256_setzero_si256();
        const __m256i one8i = _mm256_set1_epi32( 1 );
        const __m256i minone8i = _mm256_set1_epi32( -1 );

        const __m256 px8 = _mm256_set1_ps( px );
        const __m256 py8 = _mm256_set1_ps( py );

        const __m256 ol = onLeft8( vx, vy, wx, wy, px8, py8 );
        const __m256i olltz = _mm256_cmp_ps( ol, zero8, _CMP_LT_OS );
        const __m256i olgtz = _mm256_cmp_ps( ol, zero8, _CMP_GT_OS );

        const __m256i pred0 = _mm256_cmp_ps( wy, py8, _CMP_GT_OS );
        const __m256i pred1 = _mm256_cmp_ps( wy, py8, _CMP_LE_OS );
        const __m256i useup = _mm256_cmp_ps( vy, py8, _CMP_LE_OS );

        const __m256i up_intersect = _mm256_blendv_ps( zero8i,    one8i, _mm256_and_ps( pred0, olgtz ) );
        const __m256i dn_intersect = _mm256_blendv_ps( zero8i, minone8i, _mm256_and_ps( pred1, olltz ) );

        __m256i addition = _mm256_blendv_ps( dn_intersect, up_intersect, useup );

        // There should be a faster way to sum over the 8 components of a vector value.
        int additionvals[ 8 ] __attribute__ ((aligned (32)));
         _mm256_store_si256( (__m256i*) additionvals, addition );
        return additionvals[0] + additionvals[1] + additionvals[2] + additionvals[3] +
               additionvals[4] + additionvals[5] + additionvals[6] + additionvals[7];
}
```

Also, I implemented an 8-Way SIMD algorithm to calculate edge intersections.
A single edge is tested against 8 others in a single pass, no branching.

By only using 8 sided polygon shapes (either convex or concave) the overlap test becomes very efficient thanks to the parallel AVX instructions.

## Links to External Libraries
No external libraries are required.

You can use any SVG enabled browser to view the output from the executable compiled with clang.
[SVG Support](http://caniuse.com/svg "SVG Support")

## Images & Videos

![Example Image](project_images/sample5_sm.png?raw=true "Hue per shape, Saturation and Value fall off from centre. (c=1.278)")

![Example Image](project_images/sample10_sm.png?raw=true "Hue as function of distance to centre. Value/Saturation fixed. (c=1.31)")

![Example Image](project_images/sample7_sm.png?raw=true "Hue per shape" )

![Example Image](project_images/sample6_sm.png?raw=true "Single shape, single hue. Saturation and Value fall off from centre." )

