# Pathlib
- Pathlib is a module part of the standard library which contains a Path class for turning file paths into objects 

- It is imported with “From pathlib import Path”
With this Path class, file paths are objects of the respective class, as opposed to strings with the previous os.path

- This allows for class methods to be called in place on the directory Path object, shortening code as opposed to using more verbose string operations on a string path

- Use “ / “ to join paths ( “( path / ‘subdir’ / ‘file.txt’)” )

- New inspection methods are available to use on a Path object such as path.exists()

- Parsing methods are also simpler and more concise 

Inspection Methods: 

    .name (The final component of the path, “file.txt”)

	.stem  (The name without its suffix, “file”)

	.suffix (The suffix of the file name, “.txt”)

	.suffixes (List of multiple extensions, archive.tar.gz = “[.tar, .gz]”)

	.parent (The path’s parent directory as a new Path object)

	.parts (A tuple giving access to the path’s components)

	.anchor (The root of the path, “/” or “C:\”)

Action Methods:
 
    .exists() (Boolean)

	.is_file() (Boolean)

	.is_dir() (Boolean)

	.is_absolute() (Boolean)

	.stat() (Returns info)

    .read_text(encoding=None, ‘utf-8’) (Opens text file, returning it as string and closing it)

    .write_text(data, encoding=None, ‘utf-8’) (Opens text file, writes string into it and closes it)

    .read_bytes() (Opens file in bytes mode, returns contents, and closes)

    .write_bytes(data) (Opens file in bytes mode and writes bytes data into it)

    .mkdir(mode=0o777, parents=False, exist_ok=True) (Create Directory)

    .touch(mode=0o666, exist_ok=True) (Create empty file at this path)

    .rename(target) (rename path to given path)
    etc.


