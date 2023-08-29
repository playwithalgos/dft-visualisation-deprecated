<script>
    const cos = Math.cos;
    const sin = Math.sin;
    const PI = Math.PI;
    const prec = 1;

    let spectrum = [
        [0, 3],
        [6, 0],
        [0, -5],
        [2, 0],
        [2, 1],
        [0, 3],
        [0.0, 1],
        [0.0, 1],
        ...Array(100).fill([0, 0]),
    ];

    let points = [];

    let colors = [
        "black",
        "brown",
        "red",
        "orange",
        "rgb(192, 128, 0)",
        "green",
        "rgb(0, 128, 128)",
        "blue",
        "purple",
    ];

    function getColor(i) {
        return colors[i % colors.length];
    }

    function dft(points, sign, factor) {
        const spectrum = [];
        const N = points.length;
        for (let freq = 0; freq < N; freq++) {
            const c = [0, 0];

            for (let j = 0; j < N; j++) {
                c[0] +=
                    points[j][0] * cos((sign * 2 * PI * j * freq) / N) -
                    points[j][1] * sin((sign * 2 * PI * j * freq) / N);
                c[1] +=
                    points[j][0] * sin((sign * 2 * PI * j * freq) / N) +
                    points[j][1] * cos((sign * 2 * PI * j * freq) / N);
            }
            spectrum.push([c[0] * factor, c[1] * factor]);
        }
        return spectrum;
    }

    function contrib(spectrum, t, j) {
        const N = spectrum.length;
        return [
            spectrum[j][0] * cos((2 * PI * j * t) / N) -
                spectrum[j][1] * sin((2 * PI * j * t) / N),
            spectrum[j][0] * sin((2 * PI * j * t) / N) +
                spectrum[j][1] * cos((2 * PI * j * t) / N),
        ];
    }
    function point(spectrum, t, i) {
        const c = [0, 0];
        const N = spectrum.length;
        if (i == undefined) i = N;
        for (let j = 0; j < i; j++) {
            const A = contrib(spectrum, t, j);
            c[0] += A[0];
            c[1] += A[1];
        }
        return c;
    }

    let t = 0;

    setInterval(() => {
        t += 1 / prec;
    }, 30);

    function norm(v) {
        return Math.sqrt(v[0] ** 2 + v[1] ** 2);
    }

    function widthSpectrum(s) {
        return Math.max(16, norm(s) * 8);
    }

    function realWidthSpectrum(s) {
        return Math.max(16, norm(s) * 8) / 8;
    }
</script>

<h1>DFT visualisation</h1>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<svg
    id="main"
    width="480px"
    height="480px"
    viewBox="-32 -32 64 64"
    on:mousedown={() => {
        points = [];
        spectrum = [];
    }}
    on:mousemove={(evt) => {
        if (evt.buttons == 0) return;

        // Create an SVGPoint for future math
        const ptScreen = { x: evt.offsetX, y: evt.offsetY };

        const p = {
            x: ((ptScreen.x - 240) * 64) / 480,
            y: ((ptScreen.y - 240) * 64) / 480,
        };

        /* const lastpoints = points[points.length - 1];
        console.log(points.length);
        if (lastpoints) {
            const bar = (A, B, i) => A * (1 - i) + B * i;

            for (let i = 0; i <= 1; i += 0.2) {
                points.push([
                    bar(lastpoints[0], p.x, i),
                    bar(lastpoints[1], p.y, i),
                ]);
            }
        } else */
        points.push([p.x, p.y]);

        spectrum = dft(points, -1, 1 / points.length);
        const max = Math.max(...spectrum.map((s) => norm(s)));
        for (let i = 0; i < spectrum.length; i++)
            if (norm(spectrum[i]) < max / 50) spectrum[i] = [0, 0];

        //if(spectrum.length - 1 > 8)
        points = points; //dft(spectrum, 1, 1);
    }}
>
    {#each spectrum as s, i}
        <circle
            cx={point(spectrum, t, i)[0]}
            cy={point(spectrum, t, i)[1]}
            class="point"
            fill={getColor(i)}
            fill-opacity="0.2"
        />
        <circle
            cx={point(spectrum, t, i)[0]}
            cy={point(spectrum, t, i)[1]}
            r={norm(s)}
            fill-opacity="0.1"
            fill={getColor(i)}
        />
        <circle
            cx={point(spectrum, t, i)[0]}
            cy={point(spectrum, t, i)[1]}
            r={norm(s)}
            stroke-opacity="0.01"
            stroke={getColor(i)}
            fill="none"
        />
        <line
            x1={point(spectrum, t, i)[0]}
            y1={point(spectrum, t, i)[1]}
            x2={point(spectrum, t, i + 1)[0]}
            y2={point(spectrum, t, i + 1)[1]}
            stroke-opacity="0.8"
            stroke={getColor(i)}
        />
    {/each}
    {#each points as s, i}
        <circle cx={s[0]} cy={s[1]} class="point" fill={"red"} />
    {/each}
    <circle
        cx={point(spectrum, t)[0]}
        cy={point(spectrum, t)[1]}
        class="currentPoint"
    />
    {#each { length: prec * spectrum.length } as _, tt}
        <circle
            cx={point(spectrum, tt / prec)[0]}
            cy={point(spectrum, tt / prec)[1]}
            class="point"
            fill-opacity="0.2;"
            fill={"black"}
        />
    {/each}

    <circle
        cx={0}
        cy={0}
        class="point"
        fill={"white"}
        stroke-width="0.01"
        stroke="red"
    />
</svg>
<br />
{#each spectrum as s, i}
    <svg
        width={widthSpectrum(s)}
        height={widthSpectrum(s)}
        viewBox={`${-realWidthSpectrum(s)} ${-realWidthSpectrum(s)} ${
            2 * realWidthSpectrum(s)
        } ${2 * realWidthSpectrum(s)}`}
    >
        <circle cx={0} cy={0} r={0.02} fill={getColor(i)} fill-opacity="1" />

        <circle
            cx={0}
            cy={0}
            r={norm(s)}
            fill-opacity="0.2"
            fill={getColor(i)}
        />

        <line
            x1={0}
            y1={0}
            x2={s[0]}
            y2={s[1]}
            stroke-opacity="1"
            stroke={getColor(i)}
        />

        <line
            x1={0}
            y1={0}
            x2={contrib(spectrum, t, i)[0]}
            y2={contrib(spectrum, t, i)[1]}
            stroke-opacity="0.8"
            stroke={getColor(i)}
        />
    </svg>
{/each}

<style>
    @keyframes blinking {
        0% {
            r: 0.5px;
            fill: red;
        }
        90% {
            r: 1px;
            fill: red;
        }
        95% {
            r: 1px;
            fill: red;
        }
        100% {
            r: 0px;
            fill: red;
        }
    }
    .currentPoint {
        animation-name: blinking;
        animation-duration: 500ms;
        animation-iteration-count: infinite;
        fill: red;
        fill-opacity: 1;
        vector-effect: non-scaling-radius;
    }

    svg * {
        vector-effect: non-scaling-stroke;
        stroke-width: 1px;
    }

    .point {
        r: 0.2px;
        vector-effect: non-scaling-radius;
    }
</style>
