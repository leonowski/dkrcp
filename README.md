# dkrcp
Copy files between host's file system, containers, and images.
```
Usage: ./dkrcp.sh [OPTIONS] SOURCE [SOURCE]... TARGET 

  SOURCE - Can be either: 
             host file path     : {<relativepath>|<absolutePath>}
             image file path    : [<nameSpace>/...]{<repositoryName>:[<tagName>]|<UUID>}::{<relativepath>|<absolutePath>}
             container file path: {<containerName>|<UUID>}:{<relativepath>|<absolutePath>}
             stream             : -
  TARGET - See SOURCE.

  Copy SOURCE to TARGET.  SOURCE or TARGET must refer to either a container/image.
  <relativepath> within the context of a container/image is relative to
  container's/image's '/' (root).

OPTIONS:
    --ucpchk-reg=false        Don't pull images from registry. Limits image name
                                resolution to Docker local repository for  
                                both SOURCE and TARGET names.
    --author="",-a            Specify maintainer when target is an image.
    --change[],-c             Apply specified Dockerfile instruction(s) when
                                target is an image. see 'docker commit'
    --message="",-m           Apply commit message when target is an image.
    --help=false,-h           Don't display this help message.
    --version=false           Don't display version info.
```

Supplements [```docker cp```](https://docs.docker.com/engine/reference/commandline/cp/) by:
  * Facilitating image creation/adaptation by simply copying files to either a newly specified image or an existing one.  When copying to an existing image, its state is unaffected as copy preserves its immutability by creating a new layer.
  * Enabling the specification of mutiple copy sources, including other images, to improve operational alignment with linux [```cp -a```](https://en.wikipedia.org/wiki/Cp_%28Unix%29) and minimize layer creation when TARGET refers to an image.
 
#### Copy Semantics
Since ```dkrcp``` relies on ```docker cp``` its copy documentation describes the expected behavior of ```dkrcp``` when specifying only a single SOURCE argument.  However, the following table derived while designing dkrcp may present the semantics more clearly than the documention associated to ```docker cp```.

|         | Source File  | Source Directory | Source Directory Content | Stream |
| :--:    | :----------: | :---------------:| :---------------: | :-------: |
|Target exists as file. | Overlay Target with Source content. | Error |Error | Error |
|Target leaf does not exist but its parent directory does.| Leaf assumed a file. Copy Source contents to leaf name.| Leaf assumed directory. Create Target Directory with leaf name and copy Source "content" to it. | Identical behavior to adjacent left hand cell. | Error | 
#### Why?
  * Promotes smaller images and potentially minimizes their attack surface by selectively copying only those resources required to run the containerized application.
  * Facilitates manufacturing images by construction piplines that gradually evolve either toward or away from their reliance on Dockerfiles.
  * Encapsulates the reliance on and encoding of several Docker CLI calls to implement the desired functionality insulating automation incorporating this utility from potentially future improved support by Docker community members through dkrcp's interface.
