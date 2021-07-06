UE4 Voice Chat
=============
* 유튜브 링크: <https://youtu.be/nMNou_FEBx0/>


구문 추가
[Project File]/Source/[Project File Name]/[Project File Name].Build.cs

<pre>
<code>
PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HeadMountedDisplay", "OnlineSubsystem", "OnlineSubsystemUtils", "Voice"});

</code>
</pre>

[Project File]/Config/DefaultEngine.ini

<pre>
<code>
[OnlineSubsystem]
bHasVoiceEnabled=true

[Voice]
bEnabled=true
</code>
</pre>

[Project File]/Config/DefaultGame.ini

<pre>
<code>
[/Script/Engine.GameSession]
bRequiresPushToTalk=true
</code>
</pre>


헤더 파일에 추가
<pre>
<code>
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "GameFramework/Character.h"
#include "Voice.h"
#include "OnlineSubsystemUtils.h"
#include "Sound/SoundWaveProcedural.h"
#include "MyCharacter.generated.h"

UCLASS()
class MYPROJECT_API AMyCharacter : public ACharacter
{
	GENERATED_BODY()






protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	

	AMyCharacter();

	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* PlayerInputComponent) override;

	UPROPERTY()
		float VoiceCaptureVolume;

	UPROPERTY()
		bool VoiceCaptureTest;
	UPROPERTY()
		bool PlayVoiceCaptureFlag;

	UPROPERTY()
		FTimerHandle VoiceCaptureTickTimer;
	UPROPERTY()
		FTimerHandle PlayVoiceCaptureTimer;

	TSharedPtr<class IVoiceCapture> VoiceCapture;

	UPROPERTY()
		USoundWaveProcedural* VoiceCaptureSoundWaveProcedural;

	UPROPERTY(VisibleAnywhere, BluePrintReadOnly)
		UAudioComponent* VoiceCaptureAudioComponent;

	UPROPERTY()
		TArray<uint8> VoiceCaptureBuffer;

	UFUNCTION()
		void VoiceCaptureTick();

	UFUNCTION()
		void PlayVoiceCapture();
	
	
};

</code>
</pre>

C++ 파일에 추가
<pre>
<code>
// Fill out your copyright notice in the Description page of Project Settings.

#include "MyProject.h"
#include "MyCharacter.h"


// Sets default values
AMyCharacter::AMyCharacter()
{
 	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	VoiceCapture = FVoiceModule::Get().CreateVoiceCapture();
	VoiceCapture->Start();

	VoiceCaptureAudioComponent = CreateDefaultSubobject<UAudioComponent>(TEXT("VoiceCaptureAudioComponent"));
	VoiceCaptureAudioComponent->SetupAttachment(RootComponent);
	VoiceCaptureAudioComponent->bAutoActivate = true;
	VoiceCaptureAudioComponent->bAlwaysPlay = true;
	VoiceCaptureAudioComponent->PitchMultiplier = 0.85f;
	VoiceCaptureAudioComponent->VolumeMultiplier = 5.f;

	VoiceCaptureSoundWaveProcedural = NewObject<USoundWaveProcedural>();
	VoiceCaptureSoundWaveProcedural->SetSampleRate(16000);
	//VoiceCaptureSoundWaveProcedural->SampleRate = 16000;
	VoiceCaptureSoundWaveProcedural->NumChannels = 1;
	VoiceCaptureSoundWaveProcedural->Duration = INDEFINITELY_LOOPING_DURATION;
	VoiceCaptureSoundWaveProcedural->SoundGroup = SOUNDGROUP_Voice;
	VoiceCaptureSoundWaveProcedural->bLooping = false;
	VoiceCaptureSoundWaveProcedural->bProcedural = true;
	VoiceCaptureSoundWaveProcedural->Pitch = 0.85f;
	VoiceCaptureSoundWaveProcedural->Volume = 5.f;


}

// Called when the game starts or when spawned
void AMyCharacter::BeginPlay()
{
	Super::BeginPlay();

	GetWorldTimerManager().SetTimer(PlayVoiceCaptureTimer, this, &AMyCharacter::PlayVoiceCapture, 0.f, true, 0.f);
	
}

// Called every frame
void AMyCharacter::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	VoiceCaptureTick();

}

// Called to bind functionality to input
void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

}

void AMyCharacter::VoiceCaptureTick()
{
	if (!VoiceCapture.IsValid())	//VoiceCapture가 Vaild하지 않으면 return;
		return;

	uint32 VoiceCaptureBytesAvailable = 0;
	EVoiceCaptureState::Type CaptureState = VoiceCapture->GetCaptureState(VoiceCaptureBytesAvailable);

	VoiceCaptureBuffer.Reset();
	PlayVoiceCaptureFlag = false;

	if (CaptureState == EVoiceCaptureState::Ok && VoiceCaptureBytesAvailable > 0)
	{
		int16_t VoiceCaptureSample;
		uint32 VoiceCaptureReadBytes;
		float VoiceCaptureTotalSquared = 0;

		VoiceCaptureBuffer.SetNumUninitialized(VoiceCaptureBytesAvailable);

		VoiceCapture->GetVoiceData(VoiceCaptureBuffer.GetData(), VoiceCaptureBytesAvailable, VoiceCaptureReadBytes);

		for (uint32 i = 0; i < (VoiceCaptureReadBytes / 2); i++)
		{
			VoiceCaptureSample = (VoiceCaptureBuffer[i * 2 + 1] << 8) | VoiceCaptureBuffer[i * 2];
			VoiceCaptureTotalSquared += ((float)VoiceCaptureSample * (float)VoiceCaptureSample);
		}

		float VoiceCaptureMeanSquare = (2 * (VoiceCaptureTotalSquared / VoiceCaptureBuffer.Num()));
		float VoiceCaptureRms = FMath::Sqrt(VoiceCaptureMeanSquare);
		float VoiceCaptureFinalVolume = ((VoiceCaptureRms / 32768.0) * 200.f);

		VoiceCaptureVolume = VoiceCaptureFinalVolume;

		VoiceCaptureSoundWaveProcedural->QueueAudio(VoiceCaptureBuffer.GetData(), VoiceCaptureReadBytes);
		VoiceCaptureAudioComponent->SetSound(VoiceCaptureSoundWaveProcedural);

		PlayVoiceCaptureFlag = true;
	}



}

void AMyCharacter::PlayVoiceCapture()
{

	if (!PlayVoiceCaptureFlag)
	{
		VoiceCaptureAudioComponent->FadeOut(1.f, 0.f);
		return;
	}

	if (VoiceCaptureAudioComponent->IsPlaying())
		return;

	VoiceCaptureAudioComponent->Play();

}

</code>
</pre>
