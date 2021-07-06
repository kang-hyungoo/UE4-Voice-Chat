UE4 Voice Chat
=============

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
