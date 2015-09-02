All JavaScript code must conform to [Google's Style Guidelines](https://github.com/jscs-dev/node-jscs/blob/master/presets/google.json), which we enforce both at compile time (`grunt build` is configured to to lint all buildable apps) and at commit time (a hook is setup to run `jscs` against all scripts).

It is possible to sidestep both of those checks. Do not.
