---
layout: post
title:  "During opengl programming, Bluescreen popped up because of Out of memory"
date:   2021-02-24 14:38:27 +0900
categories: real-time-rendering
tags: opengl bluescreen window10 ouf-of-memory nvidia dram
comments: true  
---

여느 때 처럼, [CubbyFlow](https://github.com/utilforever/cubbyflow) 프로젝트에 들어갈 Visualization 프로그램 코드의 `Refactoring`을 하고 있었다. 

XML Scene 파일을 parsing 하는 코드 부분을 손보고 있었는데, 현재 VoxScene의 소스에는 XML Node에서 정보를 긁어내어 모든 타입의 instance를 initialization 하는 code가 집약되어 있었다. VoxScene의 Scene manager 라는 역할에 부합하지 않기도 하고 파싱할 모든 class들의 초기화 코드를 뭉쳐놓으니 복잡하고 보기도 안좋았다. 

그래서 VoxScene에서는 xml node의 name 부분을 보고 어떤 type의 instance인지 파악하여 해당 타입의 LoadXMLNode method를 호출하여 각 class에서 알아서 initialization 하기로 했다. Parsing할 instance 들은 Scene mananger에서 관리해야 하므로 모두 VoxSceneObject 라는 부모 클래스로 부터 상속을 받으며 VoxSceneObject에는 LoadXMLNode와 WriteXMLNode method가 pure method로 선언되어 있다.

```c++
class VoxSceneObject : public NonCopyable
{
public:
    //! Default constructor.
    VoxSceneObject() = default;
    //! Default destructor.
    virtual ~VoxSceneObject() {};
    //! Load scene object attributes from the xml node
    virtual void LoadXMLNode(VoxScene* scene, const pugi::xml_node& node) = 0;
    //! Write this scene object attributes to the given documents.
    virtual void WriteXMLNode(pugi::xml_node& node) = 0;
};
```

여기까진 좋았는데, 각 class의 initialization 코드를 각자 LoadXMLNode 구현체에 옮기는 과정에서, `FluidRenderable`(sequential obj anim class)에서 문제가 발생했다.

```c++
void FluidRenderable::LoadXMLNode(VoxScene* scene, const pugi::xml_node& node)
{
    //! Other initialization codes...
    
    //! Loading obj files from format and count.
    auto manager = std::make_shared<GeometryCacheManager>(format, count);
    manager->SetVertexFormat(interleavingFormat);
    //! interleaving each geometry cache vertex data.
    CubbyFlow::ParallelFor(CubbyFlow::ZERO_SIZE, manager->GetNumberOfCache(), [&](size_t index) {
        const auto& cache = manager->GetGeometryCache(index);
        cache->InterleaveData(interleavingFormat);
        });
    //! Create fluid buffer from the parsed material, program and geometry cache.
    this->SetGeometryCacheManager(manager);
    this->Resize(kMaxBufferSize);
    this->AttachMaterial(material);
    this->SetModelMatrix(mvp);
}
```

FluidRenderable의 구현에서는 매 Frame마다 다음에 rendering 될 obj의 vertices와 indices data를 transfer 하는데, 이를 rendering과 transfer를 asynchronous하게 수행하여 performance를 뽑아내고 있다. 이를 구현하기 위헤 VAO, VBO 그리고 EBO 각각 2개 이상을 미리 생성하여 double-buffering 기법처럼 frame 마다 순회하는데 위의 코드에서 Resize method가 이 buffer들을 생성하고 초기화 하는 부분이다. 

Resize method에서는 각각 미리 constant로 정의된 kMaxBufferSize(2048kB) 만큼 vertex buffer와 index buffer의 data를 확보해놓는다. 
그런데 잠깐 생각을 놓았었는지 Resize method에 kMaxBufferSize를 전달해버려 VAO, VBO, EBO를 2048kB개 만큼 생성하고 각각의 buffer에 2048kB를 할당했으므로 2048kB * 2048kB * 2 만큼의 data를 GPU에 할당하려고 한 것이다.

그 결과가 아래와 같다..
```bash
PS C:\Users\sinji\Desktop\Graphics\cubbyflow\build\x64-Release\bin> .\VoxRenderer.exe
Loading C:/Users/sinji/Desktop/Graphics/cubbyflow/Experimental/Resources/VoxRenderer.xml scene
Loading mainCam done
Loading point_lights done
Loading cubeMap done
Loading irradianceMap done
Loading refractionShader done
Loading meshShader done
Loading pointSpriteShader done
Loading watermat done
Loading particlemat done
Loading obstaclemat done
Loading fluidAnim
[Type] : Error[Source] : API[ID] : 1285[Serverity] : High
[Message] : GL_OUT_OF_MEMORY error generated. Failed to allocate memory for buffer data.
[Type] : Error[Source] : API[ID] : 0[Serverity] : High
[Message] : Unknown internal debug message. The NVIDIA OpenGL driver has encountered
an out of memory error. This application might
behave inconsistently and fail.
(pid=19328 voxrenderer.exe 64bit)
[Type] : Error[Source] : API[ID] : 1285[Serverity] : High
[Message] : GL_OUT_OF_MEMORY error generated. Failed to allocate memory for buffer data.
[Type] : Error[Source] : API[ID] : 0[Serverity] : High
[Message] : Unknown internal debug message. The NVIDIA OpenGL driver has encountered
an out of memory error. This application might
behave inconsistently and fail.
(pid=19328 voxrenderer.exe 64bit)
[Type] : Error[Source] : API[ID] : 1285[Serverity] : High
[Message] : GL_OUT_OF_MEMORY error generated. Failed to allocate memory for buffer data.
[Type] : Error[Source] : API[ID] : 0[Serverity] : High
[Message] : Unknown internal debug message. The NVIDIA OpenGL driver has encountered
an out of memory error. This application might
behave inconsistently and fail.
(pid=19328 voxrenderer.exe 64bit)
```

위와 같이 무수한 Out of memory 에러를 출력하다가
![Bluescreen](https://snowapril.github.io/assets/img/post_img/2021-02-24-bluescreen.jpg)  

퍽 소리와 함께 터져버렸다..

재부팅 된 이후 바로 코드를 확인하고 이마를 탁 치며, 역시 집중력이 중요함을 깨닫으며 정리 글을 포스팅했다.