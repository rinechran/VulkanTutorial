## About

이 튜토리얼에서는 그래픽스와 [Vulkan](https://www.khronos.org/vulkan/) API 사용방법에 대해 설명합니다. Vulkan은 [Khronos group](https://www.khronos.org/)(OpenGL)에서 만든 새로운 API로 최신 그래픽 카드를 추상적인 인터페이스로 제공한다. 새로운 인터페이스를 사용하면 [OpenGL](https://en.wikipedia.org/wiki/OpenGL) 와 [Direct3D](https://en.wikipedia.org/wiki/Direct3D) API에 비해 성능이 항샹되고 드라이버에 대해 이해할수 있게 된다. Vulkan은 [Direct3D 12](https://en.wikipedia.org/wiki/Direct3D#Direct3D_12) 
와 [Metal](https://en.wikipedia.org/wiki/Metal_(API)) 사상과 비슷하지만 Linux,Android,Windows 플래폼에 대해 크로스 컴파일이 가능한 장점이 있다.

앞서 말한 장점을 이루기 위해서의 대가는 디테일한 API를 사용해야다는 점이다. 
텍스터 이미지와 버퍼 객체를 직접 메모리 관리를 해야하며 그래픽스 API에 관련된 설정을 어플리케이션에서 처음부터 설정해야한다. 이는 그래픽 드라이버의 핸들를 적게 잡는다는 의미이며, 올바른 동작을 위해 어플리케이션에 더 많은 작업을 해야 한다.

중요한점은 Vulkan의 대상이 모두를 위한 것이 아니며, 고성능 컴퓨터 그래픽에 열광하는 프로그래머를 위한 것이다. 컴퓨터 그래픽보다 게임 개발에 관심이 있다면 Vulkan은 유리하지 않으니 OpenGL이나 Direct3D를 고수하는게 좋다. 대안으로 Vulkan보다 더 높은 고수준의 API를 제공하는 [Unreal Engine](https://en.wikipedia.org/wiki/Unreal_Engine#Unreal_Engine_4)
나 [Unity](https://en.wikipedia.org/wiki/Unity_(game_engine)) 같은 엔진을 사용하는 것이다.

튜토리얼을 위해 몇 가지의 요구사항은 다음과 같다.
* 그래픽카드 및 드라이버가 Vulkan과 호환해야한다 ([NVIDIA](https://developer.nvidia.com/vulkan-driver), [AMD](http://www.amd.com/en-us/innovations/software-technologies/technologies-gaming/vulkan), [Intel](https://software.intel.com/en-us/blogs/2016/03/14/new-intel-vulkan-beta-1540204404-graphics-driver-for-windows-78110-1540))
* C++ 경험이 있어야 한다(RALL,이니셜라이저)
* 컴파일러가 C++ 17기능을 지원해야한다 (Visual Studio 2017+, GCC 7+, Or Clang 5+)
* 기존 3D 컴퓨터 그래픽 경험이 있어야한다.

이 튜토리얼은 OpenGL이나 Direct3D 개념을 가정하지 않지만 3D 컴퓨터 그래픽스 기초 지식을 요구한다. 투시 투영에 숨겨져있는 수학 지식을 설명하지 않는다. 컴퓨터 그래픽스 개념에 볼려면 [온라인 책](https://paroj.github.io/gltut/) 참고해야한다. 또한 볼만한 컴퓨터 그래픽스 레퍼런스는 다음과 같다.

* [Ray tracing in one weekend](https://github.com/petershirley/raytracinginoneweekend)
* [Physically Based Rendering book](http://www.pbr-book.org/)
* 실제 엔진에서 Vulkan이 사용되는 오픈소스 [Quake](https://github.com/Novum/vkQuake) 와 [DOOM 3](https://github.com/DustinHLand/vkDOOM3)

You can use C instead of C++ if you want, but you will have to use a different linear algebra library and you will be on your own in terms of code structuring. We will use C++ features like classes and RAII to organize logic and resource lifetimes. There is also an alternative version of this tutorial available for Rust developers.

To make it easier to follow along for developers using other programming languages, and to get some experience with the base API we'll be using the original C API to work with Vulkan. If you are using C++, however, you may prefer using the newer Vulkan-Hpp bindings that abstract some of the dirty work and help prevent certain classes of errors.


## E-book

튜토리얼을 e-book 읽고싶다면, PDF나 EPUB를 다운로드 받을수 있다.

* [EPUB](https://raw.githubusercontent.com/Overv/VulkanTutorial/master/ebook/Vulkan%20Tutorial%20en.epub)
* [PDF](https://raw.githubusercontent.com/Overv/VulkanTutorial/master/ebook/Vulkan%20Tutorial%20en.pdf)

## Tutorial structure

We'll start with an overview of how Vulkan works and the work we'll have to do to get the first triangle on the screen. The purpose of all the smaller steps will make more sense after you've understood their basic role in the whole picture. Next, we'll set up the development environment with the Vulkan SDK, the GLM library for linear algebra operations and GLFW for window creation. The tutorial will cover how to set these up on Windows with Visual Studio, and on Ubuntu Linux with GCC.

After that we'll implement all of the basic components of a Vulkan program that are necessary to render your first triangle. Each chapter will follow roughly the following structure:

Introduce a new concept and its purpose
Use all of the relevant API calls to integrate it into your program
Abstract parts of it into helper functions
Although each chapter is written as a follow-up on the previous one, it is also possible to read the chapters as standalone articles introducing a certain Vulkan feature. That means that the site is also useful as a reference. All of the Vulkan functions and types are linked to the specification, so you can click them to learn more. Vulkan is a very new API, so there may be some shortcomings in the specification itself. You are encouraged to submit feedback to this Khronos repository.

As mentioned before, the Vulkan API has a rather verbose API with many parameters to give you maximum control over the graphics hardware. This causes basic operations like creating a texture to take a lot of steps that have to be repeated every time. Therefore we'll be creating our own collection of helper functions throughout the tutorial.

Every chapter will also conclude with a link to the full code listing up to that point. You can refer to it if you have any doubts about the structure of the code, or if you're dealing with a bug and want to compare. All of the code files have been tested on graphics cards from multiple vendors to verify correctness. Each chapter also has a comment section at the end where you can ask any questions that are relevant to the specific subject matter. Please specify your platform, driver version, source code, expected behavior and actual behavior to help us help you.

This tutorial is intended to be a community effort. Vulkan is still a very new API and best practices have not really been established yet. If you have any type of feedback on the tutorial and site itself, then please don't hesitate to submit an issue or pull request to the GitHub repository. You can watch the repository to be notified of updates to the tutorial.

After you've gone through the ritual of drawing your very first Vulkan powered triangle onscreen, we'll start expanding the program to include linear transformations, textures and 3D models.

If you've played with graphics APIs before, then you'll know that there can be a lot of steps until the first geometry shows up on the screen. There are many of these initial steps in Vulkan, but you'll see that each of the individual steps is easy to understand and does not feel redundant. It's also important to keep in mind that once you have that boring looking triangle, drawing fully textured 3D models does not take that much extra work, and each step beyond that point is much more rewarding.

If you encounter any problems while following the tutorial, then first check the FAQ to see if your problem and its solution is already listed there. If you are still stuck after that, then feel free to ask for help in the comment section of the closest related chapter.

Ready to dive into the future of high performance graphics APIs? Let's go!
