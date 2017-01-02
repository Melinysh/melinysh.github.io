---
layout: post
title: "Tinkering with Bazil's Go FUSE package"
date: 2016-12-31
categories: Go, FUSE
---

After having finished my 2A academic term at UWaterloo, I've taken a few weeks off before beginning my next co-op at Shopify as a backend intern. Over this break, I finally found the time to explore a technology that has interested me over the past few months. [FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace), which stands for Filesystem in Userspace, is software that allows for non-privileged users to create userspace filesystems. Some examples of FUSE projects are [EncFS](https://en.wikipedia.org/wiki/EncFS), an encrypted virtual filesystem, and a personal favourite, [SSHFS](https://en.wikipedia.org/wiki/SSHFS) that gives access to a remote filesystem over ssh. 

I used [Bazil's Go FUSE](https://github.com/bazil/fuse) package. It seems development of it has slowed down over the past year, but the package worked great for me. From what I found, the package and its interface have evolved steadily over time, but the documentation hasn't kept up. Tommi Virtanen's [talk](https://bazil.org/talks/2013-06-10-la-gophers/) from 2013 was a helpful overview of FUSE, but the API shown has since changed.  

# A Basic Filesystem

With Bazil's FUSE package I built a very simple filesystem. It supports CRUD operations on files and directories. I hardcoded a few files and directories to get started. My sample project was based off of the [clockfs](https://github.com/bazil/fuse/blob/master/examples/clockfs/clockfs.go) example, you can check out my final version [here](https://github.com/Melinysh/fuse-example).

{% highlight Go %}
type Node struct {
	inode uint64
	name  string
}

type File struct {
	Node
	data []byte
}

type Dir struct {
	Node
	files       *[]*File
	directories *[]*Dir
}

type FS struct {
	root *Dir
}
{% endhighlight %}

Above are the core structs used. In the `Dir` struct, the pointer to the slice was required to fix a map-related crash within Bazil and the pointers within the slice fix the problem that modifications made to some `File`s obviously don't get persisted in pass-by-value methods. Almost all of the operations will be implemented by either `Dir` or `File` through Bazil's exported interfaces.

### Implementing the `Node` Interface

The most important interface in Bazil's package is the `Node` interface. This interface is the provides the core information for parts of the filesystem, in our case, `File` and `Dir`. Implementing it is straightforward, requiring only one method:

{% highlight Go %}
type Node interface {
	Attr(ctx context.Context, attr *fuse.Attr) error
}
{% endhighlight %}
For `File`, I implemented it as follows:

{% highlight Go %}
func (f *File) Attr(ctx context.Context, a *fuse.Attr) error {
	a.Inode = f.inode
	a.Mode = 0777
	a.Size = uint64(len(f.data))
	return nil
}
{% endhighlight %}

One important note here, you see it uses a `context.Context`. This is not the context package included in Go 1.7, but it actually expects `golang.org/x/net/context`. Likely because Bazil's package hasn't been updated recently.

### Reading a File

To read a file, the `File` struct implements [`HandleReadAller`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L285).

{% highlight Go %}
func (f *File) ReadAll(ctx context.Context) ([]byte, error) {
	return f.data, nil
}
{% endhighlight %}

### Editing a File

To be able to edit and write to a file, `File` implements [`HandleWriter`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L306).

{% highlight Go %}
func (f *File) Write(ctx context.Context, req *fuse.WriteRequest, resp *fuse.WriteResponse) error {
	resp.Size = len(req.Data)
	f.data = req.Data
	return nil
}
{% endhighlight %}

The important documentation note is that `resp.Size` must be set for the amount of data actually written to your internal implementation.

When I tried this in vim, I kept getting "Fsync error E667". Fsync is a library function in vim that flushes a file to disk, making sure that it is safely written. Luckily, fixing this was as easy as implementing another Bazil interface, [`NodeFsyncer`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L221).

{% highlight Go %}
func (f *File) Fsync(ctx context.Context, req *fuse.FsyncRequest) error {
	return nil
}
{% endhighlight %}

### Creating a File

This operation will run on a `Dir`. [`NodeCreater`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L197) provides a `fuse.CreateRequest` struct that contains the file metadata including the new name. 

{% highlight Go %}
func (d *Dir) Create(ctx context.Context, req *fuse.CreateRequest, resp *fuse.CreateResponse) (fs.Node, fs.Handle, error) {
	f := &File{Node: Node{name: req.Name, inode: NewInode()}}
	files := []*File{f}
	if d.files != nil {
		files = append(files, *d.files...)
	}
	d.files = &files
	return f, f, nil
}
{% endhighlight %}


### Reading a Directory

[`NodeStringLookuper`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L163) and [`HandleReadDirAller`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L289) are two single method interfaces that enable reading directories. 

{% highlight Go %}
func (d *Dir) Lookup(ctx context.Context, name string) (fs.Node, error) {
	if d.files != nil {
		for _, n := range *d.files {
			if n.name == name {
				return n, nil
			}
		}
	}
	if d.directories != nil {
		for _, n := range *d.directories {
			if n.name == name {
				return n, nil
			}
		}
	}
	return nil, fuse.ENOENT
}
{% endhighlight %}

Given a string `NodeStringLookuper`'s Lookup method provides the Node that matches that name, otherwise, return `fuse.ENOENT`. It could be either a `File` or a sub-`Dir`, so I loop through both. 

{% highlight Go %}
func (d *Dir) ReadDirAll(ctx context.Context) ([]fuse.Dirent, error) {
	var children []fuse.Dirent
	if d.files != nil {
		for _, f := range *d.files {
			children = append(children, fuse.Dirent{Inode: f.inode, Type: fuse.DT_File, Name: f.name})
		}
	}
	if d.directories != nil {
		for _, dir := range *d.directories {
			children = append(children, fuse.Dirent{Inode: dir.inode, Type: fuse.DT_Dir, Name: dir.name})
		}
	}
	return children, nil
}
{% endhighlight %}

`ReadDirAll` is the broader method. Here you create a slice of `fuse.Dirent` for all `File`s and `Dir`s in the provided `Dir`. 

### Creating a Directory

Making a new directory is very similar to creating a file, as seen above. [`NodeMkdirer`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L179) (I think you're getting the pattern now) passes the `fuse.MkdirRequest` struct with the new directory's metadata.

{% highlight Go %}
func (d *Dir) Mkdir(ctx context.Context, req *fuse.MkdirRequest) (fs.Node, error) {
	dir := &Dir{Node: Node{name: req.Name, inode: NewInode()}}
	directories := []*Dir{dir}
	if d.directories != nil {
		directories = append(*d.directories, directories...)
	}
	d.directories = &directories
	return dir, nil
}
{% endhighlight %}

### Removing Files and Directories

The last operation I'll cover is the deletion of `File`s and `Dir`s. Both are handled by the same method, from the [`NodeRemover`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L144) interface. The passed `fuse.RemoveRequest` struct has a bool, also called `Dir`, that informs you if the remove request is for a directory or not. If the request cannot be handled, a `fuse.ENOENT` will be returned. 

{% highlight Go %}
func (d *Dir) Remove(ctx context.Context, req *fuse.RemoveRequest) error {
	if req.Dir && d.directories != nil {
		newDirs := []*Dir{}
		for _, dir := range *d.directories {
			if dir.name != req.Name {
				newDirs = append(newDirs, dir)
			}
		}
		d.directories = &newDirs
		return nil
	} else if !req.Dir && *d.files != nil {
		newFiles := []*File{}
		for _, f := range *d.files {
			if f.name != req.Name {
				newFiles = append(newFiles, f)
			}
		}
		d.files = &newFiles
		return nil
	}
	return fuse.ENOENT
}
{% endhighlight %}

Here we just filter out the name provided from the existing files and directories within the provided `Dir`. 

### Starting the filesystem

The last interface to implement is the [`FS`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L38) interface on our `FS` struct. Again, just one key method. Here, we return the root `Dir`. 

{% highlight Go %}
func (f *FS) Root() (fs.Node, error) {
	return f.root, nil
}
{% endhighlight %}

To startup your new filesystem, you must first use FUSE to mount your mountpoint, then create a [`fs.Server`](https://github.com/bazil/fuse/blob/master/fs/serve.go#L363) that serves an instance of your filesystem. In this case, a `FS` instance.

{% highlight Go %}
c, err := fuse.Mount(mountpoint)
if err != nil {
	log.Fatal(err)
}
defer c.Close()
if p := c.Protocol(); !p.HasInvalidate() {
	log.Panicln("kernel FUSE support is too old to have invalidations: version %v", p)
}
srv := fs.New(c, nil)
filesys := &FS{	&Dir{ /* ... */ } }
if err := srv.Serve(filesys); err != nil {
	log.Panicln(err)
}
// Check if the mount process has an error to report.
<-c.Ready
if err := c.MountError; err != nil {
	log.Panicln(err)
}
{% endhighlight %}

---

You can find the full sample project on my [Github](https://github.com/Melinysh/fuse-example). I implemented some more interfaces and added logging so I can see how my interactions with the filesystem translates into FUSE calls. You can find the GoDoc for Bazil's FUSE [here](https://godoc.org/bazil.org/fuse), but most of the interfaces mentioned above are documented in the [fs](https://godoc.org/bazil.org/fuse/fs) GoDoc.

FUSE is a really cool technology that I think is underutilized. One idea that I've been kicking around was an extension to my Hack The North project, [`nfinite.space`](https://github.com/Melinysh/nfinite.space). Ideally a FUSE integration with nfinite.space would provide a seamless way to integrate the service into everyone's filesystem. The extension could transparently manage sharded fileparts and communicate in the background with the central service. That'd be a good start.

So that's an overview of Bazil's FUSE package. It provides an easy, extensible interface for FUSE in Go. Of course, you can find FUSE bindings in many other languages. Try experimenting with FUSE in your next project!

