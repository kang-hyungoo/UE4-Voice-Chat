UE4 Voice Chat
=============

[Project File]/Config/DefaultEngine.ini

[OnlineSubsystem]

bHasVoiceEnabled=true

[Voice]

bEnabled=true


[Project File]/Config/DefaultGame.ini


[/Script/Engine.GameSession]

bRequiresPushToTalk=true

<pre>
<code>
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
</code>
</pre>
