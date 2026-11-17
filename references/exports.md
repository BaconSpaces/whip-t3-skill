# whip_core exports, by domain

Names only — a map for grepping. The authoritative signatures are in
`whip_core/types/server.lua` and `whip_core/types/client.lua`, and the guides are in
`whip_core/docs.html`. Call them as `Whip.core:Name(...)`.

Counts at whip_core 1.2.0: **383 server, 206 client.** If a name you want is not here, grep the
type files before concluding it does not exist — and if it really does not, the export belongs in
`whip_core`, not in a workaround.

## Server

**Players & characters** — `GetPlayer` `GetCharId` `IsPlayerLoaded` `IsPlayerDead` `GetPlayers`
`IsSessionCurrent` `GetPlayersInRange` `IsCharacterDeleted` `GetPlayerIdentifiers`
`GetPlayerMetadata` `SetPlayerMetadata` `GetPlayerCoords` `GetPlayerFullName` `GetCharacterNames`
`CanSeePlayerName` `SavePlayer` `KickPlayer` `LogoutPlayer` `RenameCharacter` `RevivePlayer`
`HoldRespawn` `ReleaseRespawn` `GetRespawnHold` `GetPlayerFlag` `GetPlayerFlags` `SetPlayerFlag`

**Money & banking** — `GetPlayerCash` `ChargePlayer` `RefundPlayer` `PayPlayer` `HoldMoney`
`SpendMoneyHold` `SettleMoneyHold` `GetMoneyHold` `GetBankAccount` `GetCharacterBankAccounts`
`GetDefaultBankAccount` `GetBalance` `GetJobBankAccounts` `GetBankTransactions`
`GetBankAccessList` `CanUseBankAccount` `CreateBankAccount` `CloseBankAccount`
`SetBankAccountFrozen` `GrantBankAccess` `CanGrantBankAccess` `RevokeBankAccess` `BankDeposit`
`BankWithdraw` `BankTransfer` `BankSystemDeposit` `BankSystemWithdraw` `BankSystemTransfer`

**Inventory & items** — `GetInventory` `AddItem` `RemoveItem` `HasItem` `GetItemCount`
`CanFitItem` `GetPockets` `GetPocketSlots` `GetPocketMaxWeight` `FindItem` `FindAllItems`
`FindItemByUid` `FindItemByMetadata` `GetItemAt` `SetItemMetadata` `SetHeldItem` `GetHeldItem`
`RegisterItem` `RetireItem` `UnretireItem` `RegisterItemDefaultMetadata` `RegisterUseHandler`
`RegisterContextAction` `AddItemOption` `RemoveItemOption` `GetItemSetting`
`OnItemSettingChanged` `GetItemDef` `GetItemDefs` `BlockHands` `UnblockHands` `AreHandsBlocked`
`FreeHands` `HoldProps` `DropHeldProps` `GetHeldPropsTag`

**Stores, containers & ground** — `RegisterStoreKind` `StoreMove` `StoreRead` `StoreCount`
`StoreGive` `StoreTake` `GetOpenStores` `OpenStore` `CloseStore` `EnsureContainer`
`CreateContainer` `DeleteContainer` `GetContainer` `UpdateContainer` `CreateGroundItem`
`DeleteGroundItem` `GetGroundItemsNearby` `GetGroundItem`

**Jobs** — `GetJob` `GetJobs` `GetJobRoles` `EnsureJob` `UpdateJob` `DeleteJob` `GetPlayerJob`
`GetPlayerJobs` `IsPlayerClockedOn` `GetPlayerRole` `HasJobPermission` `GetJobEmployees`
`GetClockedOnEmployees` `HirePlayer` `CanHirePlayer` `FirePlayer` `SetPlayerRole`
`CreateJobRole` `UpdateJobRole` `DeleteJobRole` `ClockOn` `ClockOff`

**Shops** — `GetShop` `GetShops` `EnsureShop` `OpenShop` `CreateShop` `DeleteShop` `UpdateShop`
`SetShopLayout` `SetShopItemPrice` `GetShopViewers` `GetShopCatalog` `AddCatalogItem`
`RemoveCatalogItem` `GetShopStock` `GetShopPrices`

**Vehicles** — `GetVehicle` `GetVehiclesOf` `GetVehiclePosition` `SetVehicleLocked`
`GiveVehicleKey` `RemoveVehicleKey` `ChangeVehicleLocks` `CountVehicleKeys` `HasVehicleKey`
`GetPlayerVehicleKeys` `RegisterIgnitionKey` `AdoptVehicle` `HoldVehicleEngine`
`ReleaseVehicleEngine` `GetVehicleEngineHold` `CreateOwnedVehicle` `CanBringOutVehicleFor`
`VehicleModelExists` `SpawnTempVehicle` `ClaimVehicle` `SetVehiclePlate` `SpawnVehicle`
`DespawnVehicle` `RetrieveVehicle` `TransferVehicle` `DeleteVehicle` `SetVehicleCustody`
`SetVehicleData` `GetVehicleData` `ClearVehicleData` `ClearAllVehicleData`
`OnVehicleDataChanged` `SetVehicleAppearance` `RestoreVehicleAppearance` `GetVehicleCatalog`
`RegisterVehicles` `GetVehicleImage` `HasVehicleImage` `SetVehiclePrices`
`IsPlayerDrivingVehicle` `IsPlayerInVehicle` `GetVehicleDriver` `GetPlayerVehicle`
`GetPlayerSeat`

**Vehicle stats & damage** — `RegisterVehicleStat` `GetVehicleStat` `GetVehicleStats`
`SetVehicleStat` `SetVehicleStats` `AddVehicleStat` `ClearVehicleStat` `HasVehicleStat`
`OnVehicleStatChanged` `PurgeVehicleStat` `ClearVehicleDamage` `IsVehicleWrecked`

**Garages** — `GetGarage` `GetGarages` `GetNearestGarage` `GetGarageVehicles` `CreateGarage`
`UpdateGarage` `DeleteGarage` `CanUseGarage` `SetVehicleGarage`

**Properties** — `GetProperty` `GetProperties` `GetPropertyAt` `GetCharacterProperties`
`HasPropertyAccess` `SetPropertyOwner` `GivePropertyKey` `RemovePropertyKey`
`GetPropertyStashes` `GetPropertyThemes` `SetPropertyTheme` `RegisterPropertyPack`
`GetPropertyPacks` `RegisterProps` `IsPlaceableProp`

**World** — `GetTime` `GetWeather` `IsTimeFrozen` `GetTimeScale` `IsBlackout` `SetTime`
`SetWeather` `SetTimeFrozen` `SetBlackout` `SetTimeScale` `SetPlayerWeatherLayer`
`ClearPlayerWeatherLayer` `GetPlayerWeatherLayers` `RegisterZone` `RemoveZone` `EnsureZone`
`DeleteZone` `UpdateZone` `IsPointInZone` `GetPlayersInZone` `GetVehiclesInZone` `GetZones`
`GetZone` `EnsureDoors` `EnsureDoor` `GetDoor` `GetDoorByPosition` `GetDoors` `CreateDoor`
`HoldDoor` `SetHeldDoorAccess` `ReleaseDoor` `GetDoorHold` `GetManagedDoors` `UpdateDoor`
`DeleteDoor` `SetDoorLocked` `IsDoorLocked` `CanUseDoor` `LinkDoors` `UnlinkDoors`
`GetDoorsNearby` `AddPoint` `UpdatePoint` `RemovePoint` `GetPoints` `ShowPropSet` `HidePropSet`
`SpawnNpc` `DespawnNpc` `SetNpcState` `GetNpc` `GetNpcs` `StartConversation`
`UpdateConversation` `EndConversation` `GetConversation`

**UI & player feedback** — `NotifyPlayer` `ShowPlayerText` `HidePlayerText` `ShowBlip`
`HideBlip` `SetPlayerWaypoint` `OpenPanel` `ClosePanel` `RegisterPanelItem`
`OpenPlayerInventory` `ClosePlayerInventory` `IsCharacterBeingInspected` `Progress` `SkillCheck`
`GetSkillCheckGames` `GetSkillCheckSettings` `RegisterAskChannel` `AnswerAsk`
`OpenCharacterEditor` `GetAppearanceChanges` `SetPlayerAppearance` `SanitizeAppearance`
`RegisterAccessorySlot` `GetAccessorySlots` `SetActiveTheme` `GetActiveTheme`
`RequestScreenshot`

**Stats & effects** — `RegisterStat` `GetStat` `SetStat` `AddStat` `GetStats` `OnStatChanged`
`ApplyEffect` `RemoveEffect` `HasEffect` `GetEffects` `GetDamageModifier` `GetMeleeModifier`

**Security & permissions** — `SecureEvent` `SecureCallback` `RegisterPermission` `HasPermission`
`GrantPermission` `RevokePermission` `GetPermissionHolders` `GetCommandAce` `HasDevTools`
`SetDeveloperMode` `TrackInvocationLock` `RecordEventRejection` `RegisterSentinelKey`
`SetSentinelPlayerKey` `SetSentinelEntityKey` `GetSentinelTruth` `ReleaseTag` `GetTagged`

**The Director** — `StartScenarioRun` `EndScenarioRun` `AddPlayerToScenarioRun`
`RemovePlayerFromScenarioRun` `GetScenarioRunVar` `SetScenarioRunVar` `SendDirectorEvent`
`SwitchScenario` `GetScenarioRun` `GetScenarioRuns` `GetPlayerScenarioRuns`
`IsPlayerInScenarioRun` `GetScenarioRunOf` `CanStartScenarioRun` `GetScenario` `GetScenarios`
`GetScenarioStats` `GetDirectorCards` `RemoveDirectorScenario` `RemoveDirectorTemplate`
`CheckDirectorScenario` `TestDirectorCard` `TestDirectorScenario` `SteerDirectorTest`
`GetDirectorTestResult` `GetDirectorThingData` `GetLoreCharacter` `GetLoreCharacters` `LoreSay`
`PayAsLore` `CheckPicture`

**Infrastructure** — `IsReady` `GetPlatform` `GetRegistrations` `HasExport` `GetErrorLog`
`LogActivity` `FlushActivityLog` `EnsureTable` `CommitWithAction` `GiveOnLogin`
`RegisterVoiceSystem` `GetVoiceSystem` `GetUpdate` `GetUpdates`

## Client

**Readiness & player** — `IsReady` `GetPlayer` `IsPlayerLoaded` `GetCharId` `GetPlayerFullName`
`IsPlayerDead` `EnterCharacter` `EditAppearance` `GetBodyState` `GetStat` `GetStats`
`OnStatChanged` `HasEffect` `GetEffects` `GetPlatform` `GetRegistrations` `HasExport`
`GetErrorLog`

**Inventory & hands** — `HasItem` `GetItemCount` `GetHeldItem` `GetInventory` `IsInventoryOpen`
`CloseInventory` `SetInventoryLocked` `GetItemDef` `GetItemDefs` `RegisterItemFlag`
`HasItemFlag` `GetItemSetting` `OnItemSettingChanged` `FreeHands` `BlockHands` `UnblockHands`
`AreHandsBlocked` `GetOwnHeldProp` `DropOwnHeldProp` `SetOwnHeldPropPose` `SetGiveModeStyle`

**UI** — `Notify` `Progress` `IsProgressActive` `CancelProgress` `InterruptProgress`
`ShowInterrupted` `Input` `Alert` `Confirm` `ShowText` `HideText` `ShowObjective`
`HideObjective` `SkillCheck` `GetSkillCheckSettings` `OpenComposer` `FocusComposer`
`CloseComposer` `IsComposerOpen` `OpenKeyboard` `IsRadialOpen` `AddAction` `UpdateAction`
`SetActionStyle` `RemoveAction` `RemoveActionGroup` `GetAction` `PlayPress` `SetScreenArea`
`SetScreenOpen` `IsAnyScreenOpen` `IsGameInputFree` `HoldCleanScreen` `ReleaseCleanScreen`
`AimDot` `GetNuiCaps` `ReadScreen` `PressScreen` `TakeScreenshot` `MeasureFrames`

**Controls & input** — `BlockControls` `UnblockControls` `HoldPlayerPed` `ReleasePlayerPed`
`IsControllerActive` `RegisterPadAction` `RemovePadAction` `SetPadActionReady`
`GetPadActionButton` `IsAcceptPressed` `IsAcceptHeld` `AcceptHint` `IsCancelPressed`
`CancelHint` `IsKeyPressed` `IsKeyHeld` `KeyHint`

**Streaming & placement** — `LoadModel` `LoadAnimDict` `LoadTextureDict` `LoadAnimSet`
`LoadPtfxAsset` `PlaceObject` `PlacePoint`

**Vehicles** — `GetVehicleId` `GetVehicleEntity` `HasVehicleKey` `GetVehicle` `ResolveVehicle`
`GetNearbyVehicles` `GetVehicleProperties` `SetVehicleProperties` `SetVehicleExtras`
`TakeVehicleControl` `HoldVehicleAppearance` `GetVehicleData` `OnVehicleDataChanged`
`WatchVehicleData` `UnwatchVehicleData` `GetVehicleStat` `HasVehicleStat` `GetVehicleStats`
`ConsumeVehicleStat` `ConsumeVehicleStats` `ReportVehicleStat` `OnVehicleStatChanged`
`WatchVehicleStat` `UnwatchVehicleStat` `SetVehicleStatWatchInterval` `SetVehicleStatDisplay`
`ClearVehicleStatDisplay` `SetVehicleStatShown` `ClearVehicleStatShown` `SetVehicleStatIcon`
`ClearVehicleStatIcon` `SetVehicleHandlingLayer` `ClearVehicleHandlingLayer` `SetVehiclePower`
`ClearVehiclePower` `SetVehicleRevRange` `IsVehicleElectric` `SetVehicleDialGear`
`GetVehicleHandlingBase` `GetVehiclePoint` `SetVehiclePointRules` `GetGarageHelperStatus`

**Jobs & shops** — `GetPlayerJobs` `GetPlayerJob` `IsPlayerClockedOn` `HasJobPermission`
`ClockOn` `ClockOff` `IsShopOpen` `GetActiveShopId`

**World** — `GetTime` `GetWeather` `IsTimeFrozen` `IsTimeOverridden` `SetWeatherLayer`
`ClearWeatherLayer` `GetWeatherLayers` `RegisterZone` `RemoveZone` `RemoveZoneGroup`
`UpdateZone` `IsPointInZone` `IsInZone` `GetZonesInside` `GetZone` `GetZonesNearby`
`IsDoorLocked` `GetDoorsNearby` `GetLoadedDoors` `GetDoor` `GetCurrentProperty`
`GetInteriorFurnishState` `IsEscorting` `IsBeingEscorted` `GetEscortTarget`

**Camera** — `PlayCamera` `PlayCameraPath` `PlayCameraSequence` `OrbitCamera` `StopCamera`
`IsCameraActive`

**Emotes & voice** — `PlayEmote` `CancelEmote` `IsEmoting` `SetWalkOverride` `GetVoiceLevel`
`GetVoiceName` `SetVoiceTalking` `GetVoiceSystem`

**Callbacks, hooks & dev** — `Callback` `RegisterClientHook` `ReleaseTag` `GetTagged`
`HasDevTools` `RegisterDevPage` `RefreshDevPage` `RemoveDevPage` `RegisterDebugTool`
`RemoveDebugTool` `GetPlayerSetting` `SetPlayerSetting` `GetPlayerScenarioRuns`
`PermitNativeWeapon`
