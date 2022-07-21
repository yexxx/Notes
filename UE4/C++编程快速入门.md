# C++编程快速入门

## 编程快速入门

创建首个代码项目，并添加新的C++类。

1. 创建空白模板的c++项目
2. 通过UE4编辑器创建`Actor`子类
3. 通过`UPROPERTY`设可配置参数
   ```UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="FloatingActor")```

4. 构造函数

   ```cpp
   AFloatingActor::AFloatingActor()
    {
        // 将此Actor设为逐帧调用Tick()。如无需此功能，可关闭以提高性能。
        PrimaryActorTick.bCanEverTick = true;

        VisualMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
        VisualMesh->SetupAttachment(RootComponent);

        static ConstructorHelpers::FObjectFinder<UStaticMesh> CubeVisualAsset(TEXT("/Game/StarterContent/Shapes/Shape_Cube.Shape_Cube"));

        if (CubeVisualAsset.Succeeded())
        {
            VisualMesh->SetStaticMesh(CubeVisualAsset.Object);
            VisualMesh->SetRelativeLocation(FVector(0.0f, 0.0f, 0.0f));
        }
    }
    ```

5. 逐帧调用

   ```cpp
    void AFloatingActor::Tick(float DeltaTime)
    {
        Super::Tick(DeltaTime);

        FVector NewLocation = GetActorLocation();
        FRotator NewRotation = GetActorRotation();
        float RunningTime = GetGameTimeSinceCreation();
        float DeltaHeight = (FMath::Sin(RunningTime + DeltaTime) - FMath::Sin(RunningTime));
        NewLocation.Z += DeltaHeight * 20.0f;       //Scale our height by a factor of 20
        float DeltaRotation = DeltaTime * 20.0f;    //Rotate by 20 degrees per second
        NewRotation.Yaw += DeltaRotation;
        SetActorLocationAndRotation(NewLocation, NewRotation);
    }
    ```

## 游戏控制的摄像机

了解如何激活和切换不同的视角。

1. 在合适的位置放置`摄像机Actor`, 添加任意`Actor`并添加`摄像机Actor`
2. 创建`Actor`子类`CameraDirector`
3. 定义相机结构体

    ```cpp
    USTRUCT()
    struct FMyCamera {
        GENERATED_BODY()
        UPROPERTY(EditAnywhere)
        AActor* CameraName;
    };```

4. 逐帧定义`PlayController`的摄像机

    ```cpp
    void ACameraDirector::Tick(float DeltaTime)
    {
        Super::Tick(DeltaTime);
        const float TimeBetweenCameraChanges = 2.0f;
        const float SmoothBlendTime = 0.75f;
        TimeToNextCameraChange -= DeltaTime;
        if (TimeToNextCameraChange <= 0.0f)
        {
            TimeToNextCameraChange += TimeBetweenCameraChanges;

            // 查找处理本地玩家控制的actor。
            APlayerController* OurPlayerController = UGameplayStatics::GetPlayerController(this, 0);
            if (OurPlayerController)
            {
                for (auto camera : Cameras) {
                    if (OurPlayerController->GetViewTarget() != camera.CameraName) {
                        OurPlayerController->SetViewTarget(camera.CameraName);
                        break;
                    }
                }
            }
        }
    }
    ```

## 组件和碰撞

学习利用组件将Pawn与物理交互、使用粒子效果等方法。

首先创建和加入组件

1. 创建`Pawn`子类`CollidingPawn`
2. 添加粒子组件`UPROPERTY()
class UParticleSystemComponent* OurParticleSystem;`
3. 添加要用到的依赖头文件

    ```cpp
    #include "UObject/ConstructorHelpers.h"
    #include "Particles/ParticleSystemComponent.h"
    #include "Components/SphereComponent.h"
    #include "Camera/CameraComponent.h"
    #include "GameFramework/SpringArmComponent.h"
    ```

4. 定义根组件

    ```cpp
    USphereComponent* SphereComponent = CreateDefaultSubobject<USphereComponent>(TEXT("RootComponent"));
    RootComponent = SphereComponent;
    SphereComponent->InitSphereRadius(40.0f);
    SphereComponent->SetCollisionProfileName(TEXT("Pawn"));
    ```

5. 定义网格体组件

    ```cpp
    UStaticMeshComponent* SphereVisual = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("VisualRepresentation"));
    SphereVisual->SetupAttachment(RootComponent);
    static ConstructorHelpers::FObjectFinder<UStaticMesh> SphereVisualAsset(TEXT("/Game/StarterContent/Shapes/Shape_Sphere.Shape_Sphere"));
    if (SphereVisualAsset.Succeeded())
    {
        SphereVisual->SetStaticMesh(SphereVisualAsset.Object);
        SphereVisual->SetRelativeLocation(FVector(0.0f, 0.0f, -40.0f));
        SphereVisual->SetWorldScale3D(FVector(0.8f));
    }
    ```

6. 定义粒子系统

    ```cpp
    OurParticleSystem = CreateDefaultSubobject<UParticleSystemComponent>(TEXT("MovementParticles"));
    OurParticleSystem->SetupAttachment(SphereVisual);
    OurParticleSystem->bAutoActivate = false;
    OurParticleSystem->SetRelativeLocation(FVector(-20.0f, 0.0f, 20.0f));
    static ConstructorHelpers::FObjectFinder<UParticleSystem> ParticleAsset(TEXT("/Game/StarterContent/Particles/P_Fire.P_Fire"));
    if (ParticleAsset.Succeeded())
    {
        OurParticleSystem->SetTemplate(ParticleAsset.Object);
    }
    ```

7. 弹簧臂组件

    ```cpp
    USpringArmComponent* SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraAttachmentArm"));
    SpringArm->SetupAttachment(RootComponent);
    SpringArm->SetRelativeRotation(FRotator(-45.f, 0.f, 0.f));
    SpringArm->TargetArmLength = 400.0f;
    SpringArm->bEnableCameraLag = true;
    SpringArm->CameraLagSpeed = 3.0f;
    ```

8. 摄像机组件

    ```cpp
    UCameraComponent* Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("ActualCamera"));
    Camera->SetupAttachment(SpringArm, USpringArmComponent::SocketName);
    ```

9. 设置默认控制玩家
    `AutoPossessPlayer = EAutoReceiveInput::Player0;`

然后配置输入和创建Pawn移动组件

1. 在项目设置中配置输入

    操作映射| | |
    -|-|-
    ParticleToggle|空格
    轴映射|
    MoveForward|W|1.0
    | |S|-1.0
    MoveRight|A|-1.0
    ||D|1.0
    Turn|Mouse X|1.0

2. 添加`PawnMovementComponent`子类`CollidingPawnMovementComponent`

3. 重载`TickComponent`函数

    ```cpp
    public:
        virtual void TickComponent(float DeltaTime, enum ELevelTick TickType, FActorComponentTickFunction *ThisTickFunction) override
    
    void UCollidingPawnMovementComponent::TickComponent(float DeltaTime, enum ELevelTick TickType, FActorComponentTickFunction *ThisTickFunction)
    {
        Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

        // 确保所有事物持续有效，以便进行移动。
        if (!PawnOwner || !UpdatedComponent || ShouldSkipUpdate(DeltaTime))
        {
            return;
        }

        // 获取（然后清除）ACollidingPawn::Tick中设置的移动向量
        FVector DesiredMovementThisFrame = ConsumeInputVector().GetClampedToMaxSize(1.0f) * DeltaTime * 150.0f;
        if (!DesiredMovementThisFrame.IsNearlyZero())
        {
            FHitResult Hit;
            SafeMoveUpdatedComponent(DesiredMovementThisFrame, UpdatedComponent->GetComponentRotation(), true, Hit);

            // 若发生碰撞，尝试滑过去
            if (Hit.IsValidBlockingHit())
            {
                SlideAlongSurface(DesiredMovementThisFrame, 1.f - Hit.Time, Hit.Normal, Hit);
            }
        }
    };
    ```

最后在Pawn代码中加入自定义的移动组件

1. 创建移动组件的实例，并要求其更新根(先在`.h文件`中定义`OurMovementComponent`)

    ```cpp
    OurMovementComponent = CreateDefaultSubobject<UCollidingPawnMovementComponent>(TEXT("CustomMovementComponent"));
    OurMovementComponent->UpdatedComponent = RootComponent;
    ```

2. 重载`GetMovementComponent`函数

    ```cpp
    virtual UPawnMovementComponent* GetMovementComponent() const override;

    UPawnMovementComponent* ACollidingPawn::GetMovementComponent() const
    {
        return OurMovementComponent;
    }
    ```

3. 定义按键行为

    ```cpp
    void MoveForward(float AxisValue);
    void MoveRight(float AxisValue);
    void Turn(float AxisValue);
    void ParticleToggle();

    void ACollidingPawn::MoveForward(float AxisValue)
    {
        if (OurMovementComponent && (OurMovementComponent->UpdatedComponent == RootComponent))
        {
            OurMovementComponent->AddInputVector(GetActorForwardVector() * AxisValue);
        }
    }

    void ACollidingPawn::MoveRight(float AxisValue)
    {
        if (OurMovementComponent && (OurMovementComponent->UpdatedComponent == RootComponent))
        {
            OurMovementComponent->AddInputVector(GetActorRightVector() * AxisValue);
        }
    }

    void ACollidingPawn::Turn(float AxisValue)
    {
        FRotator NewRotation = GetActorRotation();
        NewRotation.Yaw += AxisValue;
        SetActorRotation(NewRotation);
    }

    void ACollidingPawn::ParticleToggle()
    {
        if (OurParticleSystem && OurParticleSystem->Template)
        {
            OurParticleSystem->ToggleActive();
        }
    }
    ```

4. 绑定按键

    ```cpp
    void ACollidingPawn::SetupPlayerInputComponent(UInputComponent* InInputComponent)
    {
        Super::SetupPlayerInputComponent(InInputComponent);

        InInputComponent->BindAction("ParticleToggle", IE_Pressed, this, &ACollidingPawn::ParticleToggle);

        InInputComponent->BindAxis("MoveForward", this, &ACollidingPawn::MoveForward);
        InInputComponent->BindAxis("MoveRight", this, &ACollidingPawn::MoveRight);
        InInputComponent->BindAxis("Turn", this, &ACollidingPawn::Turn);
    }
    ```
