## apr 18 

[1] finding out the "what"s of the field

- used [opus 3.7](prompts/what.txt) to understand all the layers in this field, and how i might enter them.
- we have got these layers: hardware and silicon, kernels and low level compute, frameworks and model runtime, inference engines, distributed orchestration, serving infra. special mention: model optimization techniques.
- detailed docs [here](knowledge/layers.md)

[2] analysed meeting notes to get "clarity"

- read about the current infrastructure, and the intended improvements
- filtered out the immediate work that is to be done to setup the test-bed for further experiments.
- analysed the exisiting infrastructure's status and generated diagrams and a beginner manual - using claude code
- generated the list of "problems" that we need to tackle initially
- generated a manual for setting up the test-bed, on which further experiments will take place [0env.md](logs/0env.md)
- summarised everything upto this [understanding WHAT we have + minimal setup] in [stage0.md](summary/stage0.md)

## apr 19

- cleared all referencs of Ada cluster
- got an idea of what the first work will be: env setup + scripts + dockerfiles
- decide the tooling and inform supervisor [conda, ]
- omw to read:
    + [How GPU Computing Works?](https://www.youtube.com/live/3l10o0DYJXg) by Stephen Jones
    + [Internals of CPU and GPU](https://tamal.tech/cpu-gpu-internals/)
    + [NVIDIA RTX 6000 spec sheet](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/rtx-6000/proviz-print-rtx6000-datasheet-web-2504660.pdf)
    + [Roofline: An Insightful Visual Performance Model for Floating-Point Programs and Multicore Architectures](https://people.eecs.berkeley.edu/~kubitron/cs252/handouts/papers/RooflineVyNoYellow.pdf)
    + [Making Deep Learning Go Brrrr From First Principles](https://horace.io/brrr_intro.html) by Horace He
    + Simon Boehm's [blog](https://siboehm.com/)