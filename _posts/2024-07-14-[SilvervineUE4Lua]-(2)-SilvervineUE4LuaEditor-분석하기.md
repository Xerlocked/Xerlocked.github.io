---
layout: post
title:  "[SilvervineUE4Lua] (2) SilvervineUE4LuaEditor 분석"
author: "xerlock"
tags: unreal
---

## 들어가며
실버바인의 3가지 모듈 중 처음으로 알아볼 것은 LuaEditor입니다.

가장 간단하기도 하고 맨 처음 시작이 VM(Virtual Machine)을 만들고 

사용하기 때문에 해당 모듈을 먼저 알아보겠습니다.

## 1. Build.cs

먼저 모듈의 Build.cs 파일을 살펴보겠습니다.

{% highlight cpp %}
using System.IO;
using UnrealBuildTool;

namespace UnrealBuildTool.Rules
{
	public class SilvervineUE4LuaEditor : ModuleRules
	{
		public SilvervineUE4LuaEditor(ReadOnlyTargetRules Target) : base(Target)
		{
            PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
            PrivatePCHHeaderFile = "SilvervineUE4LuaEditorPCH.h";

            PublicIncludePaths.AddRange(new string[]
            {
            });

            PrivateIncludePaths.AddRange(new string[]
            {
            });

            PublicDependencyModuleNames.AddRange(new string[]
            {
                "Core",
                "CoreUObject",
                "Engine",
                "InputCore",
                "SlateCore",
                "UnrealEd",

                "SilvervineUE4Lua",
            });
        }
	}
}
{% endhighlight %}

- PCH를 사용중에 있습니다. 파일을 열어보면 아직 따로 헤더가 추가되진 않았고, 파일만 만들어져 있습니다.
- `SilvervineUE4Lua` 모듈을 필요로 합니다.


  📌 모듈을 만들고 `UnrealBuildTool.Rules` 네임스페이스로 묶어주면, 명확하게 관리할 수 있고 UBT 가 인식하므로 해당 모듈의 빌드 설정을 올바르게 처리합니다.


## 2.LuaEditor.cpp

다음으로 LuaEditor.cpp를 살펴보겠습니다.

{% highlight cpp %}
// SilvervineUE4Lua / devCAT studio
// Copyright 2016 - 2020. Nexon Korea Corporation. All rights reserved.

#include "CoreMinimal.h"
#include "Modules/ModuleInterface.h"

#include "Modules/ModuleManager.h"


#define LOCTEXT_NAMESPACE "SilvervineUE4LuaEditor"

class FSilvervineUE4LuaEditorModule : public IModuleInterface
{
	// Begin IModuleInterface
	virtual void StartupModule() override
	{
	}

	virtual void ShutdownModule() override
	{
	}
	// End IModuleInterface
};

IMPLEMENT_MODULE(FSilvervineUE4LuaEditorModule, SilvervineUE4LuaEditor)

#undef LOCTEXT_NAMESPACE
{% endhighlight %}

- 기본형은 헤더파일과 cpp 파일로 나누어져 있는데, 따로 헤더파일 없이 cpp 파일 하나에 작성되었습니다.
- 기본 파일 구조에서 따로 코드가 작성된 것 없으므로 넘어가도록 하겠습니다.

## 3.LuaVirtualMachineFactory

`LuaVirtualMachineFactory.h` 와 `LuaVirtualMachineFactory.cpp` 를 살펴보겠습니다. 먼저 헤더파일 입니다.

{% highlight cpp %}
//
// USUE4LuaVirtualMachine을 uasset으로 만들어주는 클래스 입니다. 
//
UCLASS()
class USUE4LuaVirtualMachineFactory : public UFactory
{
	GENERATED_UCLASS_BODY()
	
public:
	// Begin UFactory Interface
	virtual bool ConfigureProperties() override;
	virtual UObject* FactoryCreateNew(UClass* Class, UObject* InParent, FName Name, EObjectFlags Flags, UObject* Context, FFeedbackContext* Warn) override;
	// End UFactory Interface

private:
	UPROPERTY()
	UClass* SelectedClass = nullptr;
};
{% endhighlight %}

- 언리얼 엔진에서는 외부의 에셋을 사용하거나 자신만의 커스텀 에셋을 생성하기 위해서 `UFactory`를 지원합니다.
- 해당 클래스에서는 `ConfigureProperties` , `FactoryCreateNew` 2개의 함수를 오버라이드 하고 있습니다.

### ConfigureProperties

> **/** Opens a dialog to configure the factory properties. Return false if user opted out of configuring properties */**

- 해당 함수의 설명을 보면 속성을 구성하기 위한 대화상자를 열고, 사용자가 선택하지 않으면 false를 반환합니다.

### FactoryCreateNew

- `UFactory` 클래스는 파일을 생성하는 2가지의 함수가 있습니다.
- 외부의 파일을 가져와서 에셋을 만들 경우는 `FactoryCreateFile`
- UObject를 구성하는 경우는 `FactoryCreateNew`
- 여기서는 `FactoryCreateNew`를 사용했습니다.

다음으로 cpp 파일을 살펴보겠습니다.

{% highlight cpp %}
// SilvervineUE4Lua / devCAT studio
// Copyright 2016 - 2020. Nexon Korea Corporation. All rights reserved.

#include "LuaVirtualMachineFactory.h"

#include "ClassViewerFilter.h"
#include "ClassViewerModule.h"
#include "Kismet2/SClassPickerDialog.h"
#include "Modules/ModuleManager.h"

#include "SilvervineUE4LuaVirtualMachine.h"


USUE4LuaVirtualMachineFactory::USUE4LuaVirtualMachineFactory(const FObjectInitializer& ObjectInitializer)
	: Super(ObjectInitializer)
{
	bCreateNew = true;
	SupportedClass = USUE4LuaVirtualMachine::StaticClass();
}

bool USUE4LuaVirtualMachineFactory::ConfigureProperties()
{
	SelectedClass = nullptr;

	FClassViewerModule& ClassViewerModule = FModuleManager::LoadModuleChecked<FClassViewerModule>("ClassViewer");

	FClassViewerInitializationOptions Options;
	Options.Mode = EClassViewerMode::ClassPicker;
	Options.DisplayMode = EClassViewerDisplayMode::ListView;

	{
		struct FFilter : IClassViewerFilter
		{
			virtual bool IsClassAllowed(const FClassViewerInitializationOptions& InInitOptions, const UClass* InClass, TSharedRef<FClassViewerFilterFuncs> InFilterFuncs) override
			{
				return InClass->IsChildOf(USUE4LuaVirtualMachine::StaticClass());
			}

			virtual bool IsUnloadedClassAllowed(const FClassViewerInitializationOptions& InInitOptions, const TSharedRef<const IUnloadedBlueprintData> InUnloadedClassData, TSharedRef<FClassViewerFilterFuncs> InFilterFuncs) override
			{
				return InUnloadedClassData->IsChildOf(USUE4LuaVirtualMachine::StaticClass());
			}
		};

		Options.ClassFilter = MakeShareable(new FFilter);
	}

	const FText TitleText = FText::FromString(TEXT("Pick Parent Class"));	// #todo: localization
	UClass* ChosenClass = NULL;
	const bool bPressedOk = SClassPickerDialog::PickClass(TitleText, Options, ChosenClass, USUE4LuaVirtualMachine::StaticClass());
	if (bPressedOk)
	{
		SelectedClass = ChosenClass;
	}

	return bPressedOk;
}

UObject* USUE4LuaVirtualMachineFactory::FactoryCreateNew(UClass* Class, UObject* InParent, FName Name, EObjectFlags Flags, UObject* Context, FFeedbackContext* Warn)
{
	return NewObject<USUE4LuaVirtualMachine>(InParent, SelectedClass, Name, Flags);
}
{% endhighlight %}

크게 세 부분으로 나눌 수 있을 것 같습니다. `생성자`, `ConfigureProperties`, `FactoryCreateNew`

이를 토대로 살펴보겠습니다.

### 생성자

{% highlight cpp %}
USUE4LuaVirtualMachineFactory::USUE4LuaVirtualMachineFactory(const FObjectInitializer& ObjectInitializer)
	: Super(ObjectInitializer)
{
	bCreateNew = true;
	SupportedClass = USUE4LuaVirtualMachine::StaticClass();
}
{% endhighlight %}

- `bCreateNew`는 `CanCreateNew` 함수에 의해 호출되는데 현재 `Factory`에서 새 객체를 처음부터 만들 수 있는 경우 true입니다.
- `SupportedClass` 는 현재 Factory 에서 생산하는 클래스를 나타냅니다. 여기는 `LuaVM` 클래스를 생산하고 있습니다.

### ConfigureProperties

{% highlight cpp %}
bool USUE4LuaVirtualMachineFactory::ConfigureProperties()
{
	SelectedClass = nullptr;

	FClassViewerModule& ClassViewerModule = FModuleManager::LoadModuleChecked<FClassViewerModule>("ClassViewer");

	FClassViewerInitializationOptions Options;
	Options.Mode = EClassViewerMode::ClassPicker;
	Options.DisplayMode = EClassViewerDisplayMode::ListView;

	{
		struct FFilter : IClassViewerFilter
		{
			virtual bool IsClassAllowed(const FClassViewerInitializationOptions& InInitOptions, const UClass* InClass, TSharedRef<FClassViewerFilterFuncs> InFilterFuncs) override
			{
				return InClass->IsChildOf(USUE4LuaVirtualMachine::StaticClass());
			}

			virtual bool IsUnloadedClassAllowed(const FClassViewerInitializationOptions& InInitOptions, const TSharedRef<const IUnloadedBlueprintData> InUnloadedClassData, TSharedRef<FClassViewerFilterFuncs> InFilterFuncs) override
			{
				return InUnloadedClassData->IsChildOf(USUE4LuaVirtualMachine::StaticClass());
			}
		};

		Options.ClassFilter = MakeShareable(new FFilter);
	}

	const FText TitleText = FText::FromString(TEXT("Pick Parent Class"));	// #todo: localization
	UClass* ChosenClass = NULL;
	const bool bPressedOk = SClassPickerDialog::PickClass(TitleText, Options, ChosenClass, USUE4LuaVirtualMachine::StaticClass());
	if (bPressedOk)
	{
		SelectedClass = ChosenClass;
	}

	return bPressedOk;
}
{% endhighlight %}

- 모듈 매니저를 통해 `ClassViewerModule` 을 로드합니다.
- 이후 클래스 관련 옵션들을 설정해줍니다.
- ClassViewerFilter를 설정해줍니다. 이때 필터링하는 클래스는 `LuaVM`입니다.
- 5.0 버전부터는 `ClassFilter`가 아닌 `ClassFilters` 배열을 통해 처리를 권장합니다.
- 클래스 선택 다이얼로그를 띄우고 클래스를 선택합니다.

### FactoryCreateNew

{% highlight cpp %}
UObject* USUE4LuaVirtualMachineFactory::FactoryCreateNew(UClass* Class, UObject* InParent, FName Name, EObjectFlags Flags, UObject* Context, FFeedbackContext* Warn)
{
	return NewObject<USUE4LuaVirtualMachine>(InParent, SelectedClass, Name, Flags);
}
{% endhighlight %}

- 새로 생성된 LuaVM 을 반환합니다.

## 마무리

LuaEditor 모듈에서는 Lua 스크립트 처리에 필요한 VM을 생성하는 코드를 포함하고 있었습니다.

- UFactory로 UObject 생성하기
- 모듈매니저를 통해 ClassViewerModule를 불러오기
- 불러온 ClassViewrModule을 통해 VM 클래스 생성하기

위의 3단계를 거쳐서 VM을 생성하는 것을 알 수 있었습니다.

다음에는 본격적인 SlivervineUE4Lua를 살펴보겠습니다. 감사합니다.