# Repo Analysis

## Structure

The project is poorly structured. It contains both the web app and the transcription logic, but this does not become clear from the project structure.
Also, there is a "src" folder which contains some helper stuff, but the main logic is on root level in worker.py.
Generally, a proper package structure would be preferable, e.g. src/transcribo with the subpackages src/transcribo/web and src/transcribo/worker.
And a dedicated "deploy" folder for the Dockerfiles, bat files etc. Also, "docker_aargau" is a very confusing name... looks super unclean.
Two dependency management systems: requirements.txt and pyproject.toml. This should be avoided, as both need to be maintained and it's not necessary to use two. I would only use uv.
data contains a Python file, const.py, which seems a bit ugly, especially since it only contains data structures. It would make more sense to use separate .json or .yaml files. This would also help to make the different constants more visible ("data_leaks", "languages"). I would even question whether they are constants at all; at least the first one ("data_leaks") does not look like a constant to me...
Hmm, why is the logo in data? The logo is more like an asset. I consider data to be data that is relevant for the logic.
## Generally

No CI/CD files.
No linting/formatting files/ no automatic running on them.
No AI coding files.
The project is not up to date. A quick review with Claude showed that the project contains workarounds for the hotwords logic, which has since been resolved in the dependencies. So an update to WhisperX >= 3.4.0 (the first version that passes the hotwords on to faster-whisper; updating faster-whisper alone is not enough) would resolve them and make some of the "custom" code unnecessary, which should be done. Extending/overwriting dependency code is risky. And for good maintainability, less custom code is almost always better, especially when one already depends on these libraries.
No types, no classes, random stuff together in the same file...
No tests.

## transcribe_file (worker.py 71-193)
Too big to be one method.
Too much complexity (5-6 different if/else and try/except blocks...) -> should definitely be split into methods.
No typing, see the method arguments. -> I would always use types, as it is safer and more explicit.

Some variables are defined too early: estimated_time is probably mainly relevant for the "# Estimate run time" block. A clear indicator that this method does more than one thing.
Hmm, some magic numbers. Why is there a time.sleep(2)? First, a random sleep is almost always an indicator of bad code. I assume we have to wait until something is ready or so, but then this should rather be made explicit. Also, there should probably be an explanation of why 2 is the right value.
"time_estimate": hmm, I don't like methods that return multiple values, especially when working untyped. Rather use a dedicated object/data type for that if the two values belong together. In this example, run_time is not the responsibility of time_estimate.
Reacting to -1 values to detect failures is a bad pattern. Fail fast and react to failures of the methods, or use a concrete success state. -1 as a run_time or estimated_time value is just super arbitrary.
Hmm, the try/except and the -1 check are super confusing and unnecessary. The except part in the current implementation will almost never trigger, as errors are caught inside the method and converted into -1 values. It only catches errors raised by report_error or time.sleep inside the try block.
Why are we using os.system ffmpeg commands instead of the Python package for that?
Very bad pattern of having way too many exit points from the function, with so many unnecessary try/if blocks to end early.
