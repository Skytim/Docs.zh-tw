---
title: "ASP.NET 核心中的資料保護的非 DI 注意案例"
author: rick-anderson
description: "了解如何支援資料的保護案例，您無法或不想要使用相依性插入所提供的服務。"
manager: wpickett
ms.author: riande
ms.date: 10/14/2016
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: security/data-protection/configuration/non-di-scenarios
ms.openlocfilehash: d878bd20489876f919f2a8e0149f3f000cbf72d8
ms.sourcegitcommit: 7ac15eaae20b6d70e65f3650af050a7880115cbf
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/02/2018
---
# <a name="non-di-aware-scenarios-for-data-protection-in-aspnet-core"></a>ASP.NET 核心中的資料保護的非 DI 注意案例

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

通常是 ASP.NET Core 資料保護系統[加入至服務容器](xref:security/data-protection/consumer-apis/overview)且由透過相依性插入 (DI) 的相依元件。 不過，有一些情況下，這不可行或您想要尤其是系統匯入現有的應用程式。

若要支援這些案例中， [Microsoft.AspNetCore.DataProtection.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Extensions/)套件提供具象型別， [DataProtectionProvider](/dotnet/api/Microsoft.AspNetCore.DataProtection.DataProtectionProvider)，其中提供簡單的方式來使用資料保護不需依賴 DI。 `DataProtectionProvider`型別會實作[IDataProtectionProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.idataprotectionprovider)。 建構`DataProtectionProvider`只需要提供[DirectoryInfo](/dotnet/api/system.io.directoryinfo)執行個體，以指出提供者的密碼編譯金鑰應該儲存的位置，如下列程式碼範例所示：

[!code-none[](non-di-scenarios/_static/nodisample1.cs)]

根據預設，`DataProtectionProvider`具象型別不會加密金鑰的未經處理資料之前將它保存到檔案系統。 這是為了支援的案例，開發人員會指向網路共用和資料保護系統無法自動推算在其餘的適當的金鑰加密機制。

此外，`DataProtectionProvider`具象型別不[隔離應用程式](xref:security/data-protection/configuration/overview#per-application-isolation)預設。 使用相同的索引鍵目錄的所有應用程式可以共用裝載只要其[用途參數](xref:security/data-protection/consumer-apis/purpose-strings)比對。

[DataProtectionProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionprovider)建構函式接受可用於調整的系統行為的選擇性設定回撥。 下列範例示範如何還原隔離的明確呼叫[SetApplicationName](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.setapplicationname)。 此範例也示範如何設定系統來自動加密使用 Windows DPAPI 保存的金鑰。 如果目錄指向 UNC 共用，您可能想所有相關的電腦上發佈共用的憑證並將系統設定為使用憑證為基礎的加密功能呼叫[ProtectKeysWithCertificate](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.protectkeyswithcertificate)。

[!code-none[](non-di-scenarios/_static/nodisample2.cs)]

> [!TIP]
> 執行個體`DataProtectionProvider`具象型別很難建立。 如果應用程式會維護此類型的多個執行個體，而且它們使用相同的金鑰儲存目錄，可能會降低應用程式效能。 如果您使用`DataProtectionProvider`類型，我們建議您一次建立此類型，並重複使用它盡量。 `DataProtectionProvider`類型及其所有[IDataProtector](/dotnet/api/microsoft.aspnetcore.dataprotection.idataprotector)從它建立執行個體是安全執行緒的多個呼叫端。
