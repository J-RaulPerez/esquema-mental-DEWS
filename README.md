flowchart TB
  %% ========== BOOTSTRAP ==========
  subgraph BOOTSTRAP[bootstrap/]
    BAPP[bootstrap/app.php]
    BPROV[bootstrap/providers.php]
    BCACHE[bootstrap/cache/\npackages.php\nservices.php]
  end

  %% ========== CONFIG ==========
  subgraph CONFIG[config/]
    CAPP[app.php\n(providers, aliases)]
    CAUTH[auth.php]
    CDB[database.php]
    CFS[filesystems.php]
    CLOG[logging.php]
    CQUEUE[queue.php]
    CSESS[session.php]
    CTELES[telescope.php]
  end

  %% ========== ROUTES ==========
  subgraph ROUTES[routes/]
    RWEB[web.php]
    RCON[console.php]
  end

  %% ========== HTTP LAYER ==========
  subgraph HTTP[app/Http/]
    PC[Controllers/PageController.php]
    IC[Controllers/InventionController.php]
    MC[Controllers/MaterialController.php]
    TC[Controllers/TechnologyController.php]

    SIR[Requests/StoreInventionRequest.php]
    UIR[Requests/UpdateInventionRequest.php]
    SMR[Requests/StoreMaterialRequest.php]
    UMR[Requests/UpdateMaterialRequest.php]
    STR[Requests/StoreTechnologyRequest.php]
    UTR[Requests/UpdateTechnologyRequest.php]
  end

  %% ========== DOMAIN ==========
  subgraph SERVICES[app/Services/]
    IS[InventionService.php]
    MS[MaterialService.php]
    TS[TechnologyService.php]
    CS[CategoryService.php]
    AS[AttributeService.php]
    PS[ProjectService.php]
    RS[RecipeService.php]
    US[UserService.php]

    SDI[Contracts/*DataInterface.php]
  end

  subgraph REPOS[app/Repositories/]
    RI[Contracts/*RepositoryInterface.php]
    RE[ Eloquent/*Repository.php ]
    RC[Concerns/ChecksExistenceInProject.php]
  end

  subgraph MODELS[app/Models/]
    MI[Invention.php]
    MM[Material.php]
    MT[Technology.php]
    MCat[Category.php]
    MAtt[Attribute.php]
    MImg[Image.php]
    MProj[Project.php]
    MRec[Recipe.php]
    MUser[User.php]
    MCon[Concerns/\nHasNormalizedName\nHasFormattedDescription\nHasModelType]
  end

  %% ========== EVENTS / LISTENERS ==========
  subgraph EVENTS[app/Events/]
    EIC[InventionCreated.php]
    EIU[InventionUpdated.php]
    EID[InventionDeleted.php]
    EMC[MaterialCreated.php]
    EMU[MaterialUpdated.php]
    EMD[MaterialDeleted.php]
    ETC[TechnologyCreated.php]
    ETU[TechnologyUpdated.php]
    ETD[TechnologyDeleted.php]
  end

  subgraph LISTENERS[app/Listeners/]
    LIC[LogInventionCreation.php]
    LIU[LogInventionUpdate.php]
    LID[LogInventionDeletion.php]
    LMC[LogMaterialCreation.php]
    LMU[LogMaterialUpdate.php]
    LMD[LogMaterialDeletion.php]
    LTC[LogTechnologyCreation.php]
    LTU[LogTechnologyUpdate.php]
    LTD[LogTechnologyDeletion.php]
  end

  %% ========== PROVIDERS ==========
  subgraph PROVIDERS[app/Providers/]
    ASP[AppServiceProvider.php]
    DSP[DataServiceProvider.php\n(binds interfaces -> repos)]
    TSP[TelescopeServiceProvider.php]
  end

  %% ========== VIEW / UI ==========
  subgraph VIEWMODELS[app/ViewModels/]
    BVM[BaseViewModel.php]
    IVM[InventionViewModel.php]
    MVM[MaterialViewModel.php]
    TVM[TechnologyViewModel.php]
  end

  subgraph VCOMPS[app/View/Components/]
    VCFlash[FlashMessage.php]
    VCProc[ProcessStatus.php]
    VCProp[PropertyPanel.php]
    VCActions[ResourceActions.php]
    VCForm[ResourceForm.php]
    VCGen[ResourceGeneralInfo.php]
    VCTable[ResourceTable.php]
  end

  subgraph RESOURCES[resources/]
    RJS[js/\napp.js\nbootstrap.js]
    RCSS[css/app.css]
    subgraph VIEWS[views/]
      VW[welcome.blade.php]
      VA[about.blade.php]
      VLAY[layouts/\napp.blade.php\npartials/*]
      VCOMP[views/components/*\n(action-button, flash-message,\nresource-table, ...)]
      VRES[views/resources/\nindex/create/show/edit]
      VMP[views/materials/partials/\nproperty-groups.blade.php]
      V404[errors/404.blade.php]
      VINV[inventions/ (carpeta)]
      VTEC[technologies/ (carpeta)]
      VMAT[materials/ (carpeta)]
    end
  end

  %% ========== DATABASE ==========
  subgraph DATABASE[database/]
    DSQL[database.sqlite]
    subgraph MIG[migrations/]
      M1[create_users/cache/jobs]
      M2[create_projects/categories/attributes]
      M3[create_technologies/inventions/materials/recipes/images]
      M4[pivots:\nattribute_* + recipe_requirements]
      M5[soft_deletes to many tables]
      MTEL[create_telescope_entries_table]
    end
    subgraph SEED[seeders/]
      DS[DatabaseSeeder.php]
      SAttr[AttributeSeeder.php]
      SCat[CategorySeeder.php]
      SInv[InventionSeeder.php]
      SMat[MaterialSeeder.php]
      STec[TechnologySeeder.php]
      SImg[ImageSeeder.php]
      SProj[ProjectSeeder.php]
      SRec[RecipeSeeder.php]
      SUser[UserSeeder.php]
      SCon[Concerns/HasAttributeSeeding.php]
    end
    subgraph FACT[factories/]
      FImg[ImageFactory.php]
      FProj[ProjectFactory.php]
      FRec[RecipeFactory.php]
      FUser[UserFactory.php]
    end
    subgraph DDATA[data/]
      MD1[mock-*.php\n(inventions/materials/technologies/categories/attributes)]
    end
  end

  %% ========== FLOW: REQUEST -> RESPONSE ==========
  U[User / Browser] -->|HTTP GET/POST| PUB[public/index.php]
  PUB --> BAPP --> RWEB

  RWEB -->|"/"| PC -->|return view| VW
  RWEB -->|"/about"| PC -->|return view| VA

  RWEB -->|resource inventions| IC
  RWEB -->|resource materials| MC
  RWEB -->|resource technologies| TC

  %% Requests (validation) into controllers (writes)
  IC -->|store uses| SIR -->|validated()| IS
  IC -->|update uses| UIR -->|validated()| IS
  MC -->|store uses| SMR -->|validated()| MS
  MC -->|update uses| UMR -->|validated()| MS
  TC -->|store uses| STR -->|validated()| TS
  TC -->|update uses| UTR -->|validated()| TS

  %% Service -> Repos -> Models -> DB
  IS --> RI --> RE --> MI --> DSQL
  MS --> RI --> RE --> MM --> DSQL
  TS --> RI --> RE --> MT --> DSQL

  %% Services also touch related models
  IS -.-> MCat
  IS -.-> MAtt
  IS -.-> MImg
  MS -.-> MAtt
  TS -.-> MCat
  TS -.-> MAtt

  %% Providers bind interfaces
  CAPP -->|loads providers| ASP
  CAPP -->|loads providers| DSP
  DSP -->|binds| RI
  RI -->|resolved to| RE

  %% Events and listeners (side effects)
  IS -->|event| EIC --> LIC
  IS -->|event| EIU --> LIU
  IS -->|event| EID --> LID
  MS -->|event| EMC --> LMC
  MS -->|event| EMU --> LMU
  MS -->|event| EMD --> LMD
  TS -->|event| ETC --> LTC
  TS -->|event| ETU --> LTU
  TS -->|event| ETD --> LTD

  %% ViewModels + Views + Components
  IC -->|builds| IVM -->|passes data| VRES
  MC -->|builds| MVM -->|passes data| VRES
  TC -->|builds| TVM -->|passes data| VRES

  VRES --> VCOMP
  VW --> VLAY
  VA --> VLAY

  %% Assets pipeline reference
  VLAY -.-> RJS
  VLAY -.-> RCSS

  %% Database structure relationships
  MIG -->|creates tables| DSQL
  SEED -->|fills data| DSQL
  FACT -->|generates test data| DSQL
  DDATA -->|mock arrays used by seeders| SEED
