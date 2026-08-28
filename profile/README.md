# Dial-Up

A from-scratch modding framework for Rocket League / Unreal Engine 3.

### Want Dial-Up?

→ [Dial-Up](https://github.com/dialup-mods/dialup)
The complete framework, plugin system, examples, and tooling.

### Just want the SDK tooling?

→ [SDK Generator](https://github.com/dialup-mods/sdk-generator)
Generate a clean Unreal Engine SDK, then build it into a reusable library
that can be safely shared across multiple plugins.

### Synopsis

```cpp
// Create a toast notification plugin

class Toaster : public IToaster {
    AIM_INJECTABLE(Toaster)
    AIM_INJECT(IRuntime, runtime)
    AIM_INJECT(IProcessEvent, processEvent)
    AIM_INJECT(TaskBuilder, taskBuilder)

    void toast(const std::wstring& title, const std::wstring& body) override {
        processEvent_->enableTask(
            TaskBuilder()
                .functionName("Function Engine.HUD.PostRender")
                .phase(HookPhase::Post)
                .callback([runtime = runtime_, title, body](InvocationContext& ctx) mutable {
                    auto* mgr = runtime->getFirst<UNotificationManager_TA>();
                    auto* notificationClass = runtime->classOf<UGenericNotification_TA>();
                    UNotification_TA* ret = mgr->PopUpOnlyNotification(notificationClass);
                    ret->SetTitle(FString(title));
                    ret->SetBody(FString(body));
                })
                .once()
                .build());
    }
};

class ToastPlugin : public PluginBase<ToastPlugin> {
    void startup() override {
        registerModule(ModuleDefinition<Toaster>()
            .withDependency(&Toaster::__inject_runtime, "[default]")
            .withDependency(&Toaster::__inject_processEvent, "[default]")
            .asSingleton()
        );
        setPluginReady();
    }

    void shutdown() override {
        setPluginYeetable();
    }

    auto registerPublicInterfaces() const -> std::vector<PublicInterface> override {
        return {
            expose<IToaster>(resolve<Toaster>())
        };
    }
};
```

```cpp
// Call `toast()` from a separate plugin

class BreakfastChef : public PluginBase<BreakfastChef> {
    void startup() override {
        setPluginReady();

        auto toaster = resolve<IToaster>();
        toaster->toast(L"Welcome.", L"You've got mail!");
    }
};
```
