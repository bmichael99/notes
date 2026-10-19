FOMS (and most of the playground) is currently built with ajax endpoints in a file called `ajax.php` in the root of the project directory.

This is a problem for a number of reasons, biggest of which is that the file is very difficult to maintain since its approaching 3k lines.

Moving to a proper Controller approach would allow us to unit test actions a lot easier to catch problems before they happen.