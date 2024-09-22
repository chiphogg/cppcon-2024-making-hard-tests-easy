# Making Hard Tests Easy: A Case Study From the Motion Planning Domain

## Talk slides from CppCon 2024

This repo contains the source code for the slides of Chip Hogg's CppCon 2024 talk, "Making Hard
Tests Easy: A Case Study From the Motion Planning Domain".  See the [abstract].

## How to view

You can [view the slides online](https://chogg.name/cppcon-2024-making-hard-tests-easy/).  Use the
typical controls for a `reveal.js` presentation.  In particular:

- `s` brings up the speaker notes
- `f` makes the presentation fullscreen
- `Esc` gives easy navigation
- `j`/`k` or arrow keys navigate the slides

### Local checkout option

This isn't necessary if you just want to view --- the above link should be enough for that.  But if
it's not working, or if you want to play around with the slides, you can follow these steps.

```sh
git clone https://github.com/chiphogg/cppcon-2024-making-hard-tests-easy.git
cd cppcon-2024-making-hard-tests-easy
git submodule init
git submodule update
python -m http.server
```

(For that last step, any local HTTP server will do.)

Then, just open up the listed URL in your browser!

[abstract]: https://cppcon2024.sched.com/event/1gZh3/making-hard-tests-easy-a-case-study-from-the-motion-planning-domain
