<script>
    const cos = Math.cos;
    const sin = Math.sin;
    const PI = Math.PI;
    const prec = 32;
    const SPEED = 0.01;

    let currentFrequency = undefined;
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
    let enabled = Array(200).fill(true);

    let frequenciesAlreadyChanged = [];

    let points = [];

    let colors = [
        "brown",
        "red",
        "orange",
        "yellow",
        "lightgreen",
        "green",
        "cyan",
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

    function contrib(spectrum, enabled, t, j) {
        const N = spectrum.length;
        const spX = enabled[j] ? spectrum[j][0] : 0;
        const spY = enabled[j] ? spectrum[j][1] : 0;
        const angle =
            j < N / 2 ? (2 * PI * j * t) / N : (-2 * PI * (N - j) * t) / N;
        return [
            spX * cos(angle) - spY * sin(angle),
            spX * sin(angle) + spY * cos(angle),
        ];
    }
    function point(spectrum, enabled, t, i) {
        const c = [0, 0];
        const N = spectrum.length;
        if (i == undefined) i = N;
        for (let j = 0; j < i; j++) {
            const A = contrib(spectrum, enabled, t, j);
            c[0] += A[0];
            c[1] += A[1];
        }
        return c;
    }

    let t = 0;

    setInterval(() => {
        t += SPEED * spectrum.length;
    }, 50);

    function norm(v) {
        return Math.sqrt(v[0] ** 2 + v[1] ** 2);
    }

    function widthSpectrum(s) {
        return Math.max(24, norm(s) * 8);
    }

    function realWidthSpectrum(s) {
        return Math.max(24, norm(s) * 8) / 8;
    }
</script>

<h1>Discrete Fourier Transform Visualisation</h1>

<span class="help">(draw below a closed curve)</span>

<br />
<!-- svelte-ignore a11y-no-static-element-interactions -->
<svg
    id="main"
    width="480px"
    height="480px"
    viewBox="-32 -32 64 64"
    on:mousedown={() => {
        points = [];
        spectrum = [];
        enabled = [];
        frequenciesAlreadyChanged = [];
    }}
    on:mousemove={(evt) => {
        if (frequenciesAlreadyChanged.length > 0) return;
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
        enabled = Array(points.length).fill(true);
        const max = Math.max(...spectrum.map((s) => norm(s)));
        for (let i = 0; i < spectrum.length; i++)
            if (norm(spectrum[i]) < max / 50) spectrum[i] = [0, 0];

        //if(spectrum.length - 1 > 8)
        points = points; //dft(spectrum, 1, 1);
    }}
    on:mouseup={() => (points = [])}
>
    {#each spectrum as s, i}
        {#if enabled[i]}
            <circle
                cx={point(spectrum, enabled, t, i)[0]}
                cy={point(spectrum, enabled, t, i)[1]}
                class="point"
                fill={getColor(i)}
                fill-opacity="0.2"
            />
            <circle
                cx={point(spectrum, enabled, t, i)[0]}
                cy={point(spectrum, enabled, t, i)[1]}
                r={norm(s)}
                class="wheel"
                opacity={currentFrequency == i ? 1 : 0.2}
                fill={getColor(i)}
            />
            <circle
                cx={point(spectrum, enabled, t, i)[0]}
                cy={point(spectrum, enabled, t, i)[1]}
                r={norm(s)}
                stroke-opacity="0.01"
                stroke={getColor(i)}
                fill="none"
            />
            <line
                x1={point(spectrum, enabled, t, i)[0]}
                y1={point(spectrum, enabled, t, i)[1]}
                x2={point(spectrum, enabled, t, i + 1)[0]}
                y2={point(spectrum, enabled, t, i + 1)[1]}
                stroke-opacity="1"
                stroke={getColor(i)}
            />
        {/if}
    {/each}

    <circle
        cx={point(spectrum, enabled, t)[0]}
        cy={point(spectrum, enabled, t)[1]}
        class="currentPoint"
    />
    {#each { length: prec * spectrum.length } as _, tt}
        <circle
            cx={point(spectrum, enabled, tt / prec)[0]}
            cy={point(spectrum, enabled, tt / prec)[1]}
            class="point"
            fill-opacity="0.9"
            fill={points.length == 0 ? "white" : "gray"}
        />
    {/each}

    {#each points as s, i}
        <circle cx={s[0]} cy={s[1]} class="point" fill={"white"} />
    {/each}

    <circle cx={0} cy={0} class="point" fill={"white"} stroke="red" />
</svg>
<br />

<!-- a comment here -->
Frequencies:
{#each spectrum as s, i}
    {#if norm(s) > 0}
        <span
            class="frequency"
            on:mouseenter={() => {
                currentFrequency = i;
            }}
            on:mouseleave={() => {
                currentFrequency = undefined;
            }}
        >
            <svg
                class="frequencySVG"
                style={enabled[i] ? "none" : "filter: grayscale(1)"}
                width={widthSpectrum(s)}
                height={widthSpectrum(s)}
                viewBox={`${-realWidthSpectrum(s)} ${-realWidthSpectrum(s)} ${
                    2 * realWidthSpectrum(s)
                } ${2 * realWidthSpectrum(s)}`}
                on:click={() => {}}
            >
                <circle
                    cx={0}
                    cy={0}
                    r={0.5}
                    fill={getColor(i)}
                    fill-opacity="1"
                />

                <circle
                    cx={0}
                    cy={0}
                    r={norm(s)}
                    fill={getColor(i)}
                    class="wheel"
                />

                <line
                    x1={0}
                    y1={0}
                    x2={s[0]}
                    y2={s[1]}
                    stroke-opacity="1"
                    stroke={"white"}
                />

                <line
                    x1={0}
                    y1={0}
                    x2={contrib(spectrum, enabled, t, i)[0]}
                    y2={contrib(spectrum, enabled, t, i)[1]}
                    stroke-opacity="0.8"
                    stroke={getColor(i)}
                />
                <text
                    stroke={enabled[i] ? "lightgray" : "white"}
                    font-size="3"
                    text-anchor="middle">{i}</text
                >
            </svg>
            <br />
            <input
                id={"inputCheckbox" + i}
                type="checkbox"
                bind:checked={enabled[i]}
            />
        </span>
    {/if}
{/each}

<br />
<span class="help">(click to disable/enable some frequencies)</span>

<style>
    @keyframes blinking {
        0% {
            r: 0.2px;
            fill: white;
        }
        90% {
            r: 1px;
            fill: white;
        }
        95% {
            r: 1px;
            fill: white;
        }
        100% {
            r: 0.2px;
            fill: white;
        }
    }
    .currentPoint {
        animation-name: blinking;
        animation-duration: 500ms;
        animation-iteration-count: infinite;
        fill: white;
        fill-opacity: 1;
        vector-effect: non-scaling-radius;
    }

    :global(body) {
        background: rgb(41, 41, 41);
        color: white;
    }

    #main {
        background: black;
    }

    svg * {
        vector-effect: non-scaling-stroke;
        stroke-width: 1px;
    }

    .point {
        r: 0.2px;
        vector-effect: non-scaling-radius;
    }

    .wheel {
        fill-opacity: 0.6;
    }

    text {
        user-select: none;
    }

    .frequencySVG {
        cursor: pointer;
        background: "none";
    }

    input {
        cursor: pointer;
    }
    .frequency {
        display: inline-block;
    }

    h1 {
        font-size: 20px;
    }

    .help {
        color: lightgoldenrodyellow;
        font-size: 12px;
    }
</style>
