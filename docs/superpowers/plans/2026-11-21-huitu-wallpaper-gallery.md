# 汇图壁纸网前端改造 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 把 Vue 前端整体改造成“汇图壁纸网”的影集画廊体验，并保留现有业务行为、接口和权限。

**Architecture:** 建立一套共享 CSS 变量、品牌 SVG 和响应式布局；公开图库使用独立画廊壳，登录后空间与管理功能使用工具壳。各页面只重做展示层，继续调用当前 Vue 组件、API client、路由和 Pinia 状态。

**Tech Stack:** Vue 3、TypeScript、Ant Design Vue、Vue Router、Vite；不添加依赖。

**Spec:** [2026-11-21-huitu-wallpaper-gallery-design.md](../specs/2026-11-21-huitu-wallpaper-gallery-design.md)

## Global Constraints

- 产品名统一为“汇图壁纸网”，并移除旧品牌名称、旧图标和旧品牌推广链接。
- 设计颜色固定为雾灰 `#F3F5F2`、画纸白 `#FFFFFF`、墨蓝 `#17252D`、海图蓝 `#315FE8`、朱橙 `#E66B4E`、雾线 `#DCE3E3`。
- 沿用 Vue 3 与 Ant Design Vue，不添加新依赖。
- 不改服务端接口、路由、数据结构、空间权限或审核/扩图/搜图/批量业务规则。
- 不创建虚构壁纸、测试账号或临时空间；空图库呈现品牌空状态。
- 页面适配桌面与手机，键盘焦点可见，遵从减少动态效果偏好。

## Review Focus

1. **旧品牌回流**（Task 1）：全前端源码、元信息、README、包名和构建产物搜索“鱼皮”“codefather”“yu-picture”，确认品牌表面没有残留。
2. **空图库展示假内容**（Task 2）：在空列表时确认只展示品牌空状态，图片数量为 0，不出现占位/样例图片。
3. **手机导航与内容溢出**（Task 1、2、3、4、5）：在 360px 宽度检查首页、详情、表单、空间和管理表格；必要时表格横向滚动，但页面本身不横向溢出。
4. **权限按钮误显**（Task 2、4、5）：未登录用户不显示编辑/删除/管理入口；详情页继续按 `permissionList` 控制操作。
5. **窄屏或长文本破坏操作布局**（Task 3、4、5）：长空间名、长标签与多操作按钮可换行/滚动，不遮挡主要动作。

---

### Task 1: 品牌、设计系统与双布局外壳

**Files:**
- Create: `yu-picture-frontend/src/styles/theme.css`
- Create: `yu-picture-frontend/src/assets/brand-mark.svg`
- Create: `yu-picture-frontend/public/favicon.svg`
- Modify: `yu-picture-frontend/src/main.ts`
- Modify: `yu-picture-frontend/index.html`
- Modify: `yu-picture-frontend/src/App.vue`
- Modify: `yu-picture-frontend/src/components/GlobalHeader.vue`
- Modify: `yu-picture-frontend/src/components/GlobalSider.vue`
- Modify: `yu-picture-frontend/src/layouts/BasicLayout.vue`
- Modify: `yu-picture-frontend/package.json`, `package-lock.json`, `README.md`
- Delete: `yu-picture-frontend/src/assets/logo.png`, `yu-picture-frontend/public/favicon.ico`

**Interfaces:**
- Produces: CSS custom properties `--brand-canvas`, `--brand-surface`, `--brand-ink`, `--brand-primary`, `--brand-accent`, `--brand-line`; global `body`, button, input, card, table and focus styles.
- Produces: `brand-mark.svg` for app header and `favicon.svg` for browser metadata.
- Consumes: current `loginUserStore`, route list, and existing permission-filtered menu items.

- [x] **Step 1: Add design tokens and reset rules** in `src/styles/theme.css`; use the six exact colors above, title font stack `Songti SC, STSong, Noto Serif CJK SC, serif`, and UI stack `PingFang SC, Noto Sans SC, Microsoft YaHei, sans-serif`.
- [x] **Step 2: Add the local H-frame brand mark and favicon**; confirm both SVGs render at 16px and 48px without external assets.
- [x] **Step 3: Import the global theme after Ant Design reset** in `src/main.ts`; keep all component library behavior intact.
- [x] **Step 4: Split `BasicLayout.vue` into public gallery and authenticated workspace presentation** based on current route and `loginUserStore`; public pages have no fixed sidebar/footer, workspace retains compact navigation and page content.
- [x] **Step 5: Restyle `GlobalHeader.vue` and `GlobalSider.vue`**; remove old brand/product links, keep existing admin menu filter and team-space loading.
- [x] **Step 6: Update `index.html`, README, and frontend package metadata** to `汇图壁纸网` / `huitu-wallpaper-web`; keep the lockfile in sync and remove the old raster/logo favicon files.
- [x] **Step 7: Verify brand cleanup** with `rg -n -i "鱼皮|codefather|yu-picture" src public index.html README.md package.json package-lock.json`; expected: no matches.
- [x] **Step 8: Run** `npm run build` from `yu-picture-frontend`; expected: type check and Vite production build pass.

### Task 2: 公共图库、图片详情与分享

**Files:**
- Modify: `yu-picture-frontend/src/pages/HomePage.vue`
- Modify: `yu-picture-frontend/src/components/PictureList.vue`
- Modify: `yu-picture-frontend/src/pages/SearchPicturePage.vue`
- Modify: `yu-picture-frontend/src/pages/PictureDetailPage.vue`
- Modify: `yu-picture-frontend/src/components/ShareModal.vue`

**Interfaces:**
- Consumes: Task 1 tokens, public layout, existing picture-list/detail/search API calls, and `permissionList`.
- Produces: responsive image-first gallery card and consistent detail/share presentation; no change to API signatures.

- [x] **Step 1: Reshape `HomePage.vue`** into an image-led masthead, search field, category/tag filters, gallery, and pager; retain `fetchData`, `doSearch`, category/tag inputs, and pagination behavior.
- [x] **Step 2: Restyle `PictureList.vue`** with aspect-ratio cards, image-forward hover treatment and compact action controls; retain click, share, search, edit, and delete handlers.
- [x] **Step 3: Add explicit loading and empty visual states** using the actual `loading` and `dataList` props; empty state contains no fabricated pictures. For each card use `picWidth / picHeight` when both values are positive, otherwise use `4 / 3` as the CSS aspect-ratio fallback.
- [x] **Step 4: Reshape `PictureDetailPage.vue`** into a large image canvas plus an information/action panel; retain download, share, edit, delete and permission checks.
- [x] **Step 5: Restyle `SearchPicturePage.vue` and `ShareModal.vue`**; remove the old domain fallback and use only the current site’s generated share link.
- [x] **Step 6: Run** `npm run type-check` and `npm run build`; expected: both pass.
- [ ] **Step 7: Browser-check** public home at desktop and 360px widths, empty gallery, one real returned image if available, detail actions, and copy/share dialog.

### Task 3: 登录、上传、空间创建与图片操作表单

**Files:**
- Modify: `yu-picture-frontend/src/pages/user/UserLoginPage.vue`
- Modify: `yu-picture-frontend/src/pages/user/UserRegisterPage.vue`
- Modify: `yu-picture-frontend/src/pages/AddPicturePage.vue`
- Modify: `yu-picture-frontend/src/pages/AddPictureBatchPage.vue`
- Modify: `yu-picture-frontend/src/pages/AddSpacePage.vue`
- Modify: `yu-picture-frontend/src/pages/UserExchangeVipPage.vue`
- Modify: `yu-picture-frontend/src/components/PictureUpload.vue`
- Modify: `yu-picture-frontend/src/components/UrlPictureUpload.vue`
- Modify: `yu-picture-frontend/src/components/ImageCropper.vue`
- Modify: `yu-picture-frontend/src/components/ImageOutPainting.vue`
- Modify: `yu-picture-frontend/src/components/BatchEditPictureModal.vue`
- Modify: `yu-picture-frontend/src/components/PictureSearchForm.vue`

**Interfaces:**
- Consumes: Task 1 tokens and existing form/request handlers.
- Produces: shared form-field, upload-dropzone, modal and progress visuals; all existing fields and handler signatures stay unchanged.

- [x] **Step 1: Restyle login and registration** with the new mark, restrained gallery imagery only when sourced from real content, and clear form/error states; retain current login/register requests and navigation.
- [x] **Step 2: Restyle upload and crop components** with a consistent dropzone, preview frame, file metadata and progress state; retain file input, crop events and request logic.
- [x] **Step 3: Restyle create-picture, batch-import and create-space pages**; retain route query handling, field validation and form submissions.
- [x] **Step 4: Restyle batch-edit, picture-search, VIP exchange and outpainting modals**; preserve action availability and status polling behavior.
- [ ] **Step 5: Check 360px forms and long labels**; buttons remain reachable and form controls do not overflow.
- [x] **Step 6: Run** `npm run type-check` and `npm run build`; expected: both pass.
- [ ] **Step 7: Browser-check** login/register, local upload form, URL upload form, batch import form and edit modal without creating records; verify outpainting UI states without submitting a paid task.

### Task 4: 空间、成员和分析工作区

**Files:**
- Modify: `yu-picture-frontend/src/pages/MySpacePage.vue`
- Modify: `yu-picture-frontend/src/pages/SpaceDetailPage.vue`
- Modify: `yu-picture-frontend/src/pages/SpaceAnalyzePage.vue`
- Modify: `yu-picture-frontend/src/components/analyze/SpaceCategoryAnalyze.vue`
- Modify: `yu-picture-frontend/src/components/analyze/SpaceRankAnalyze.vue`
- Modify: `yu-picture-frontend/src/components/analyze/SpaceSizeAnalyze.vue`
- Modify: `yu-picture-frontend/src/components/analyze/SpaceTagAnalyze.vue`
- Modify: `yu-picture-frontend/src/components/analyze/SpaceUsageAnalyze.vue`
- Modify: `yu-picture-frontend/src/components/analyze/SpaceUserAnalyze.vue`

**Interfaces:**
- Consumes: Task 1 workspace shell and current space/analysis API client.
- Produces: consistent workspace header, quota summaries, image-list toolbar and chart panels; route props and API shapes unchanged.

- [x] **Step 1: Restyle `SpaceDetailPage.vue`** with workspace title, quota summary and image toolbar; retain pagination, upload/edit controls and batch-edit selection behavior.
- [x] **Step 2: Restyle `SpaceAnalyzePage.vue` and all six analysis components**; keep ECharts data and requests unchanged, use shared semantic colors and responsive chart containers.
- [x] **Step 3: Restyle team-member management** with clear member roles and actions; preserve permission-specific controls.
- [x] **Step 4: Keep `MySpacePage.vue` redirect behavior** while replacing its placeholder with a branded loading state.
- [ ] **Step 5: Browser-check** a logged-in owner/editor/viewer path when the real account is available; verify unauthorized controls remain hidden and charts fit a narrow screen.
- [x] **Step 6: Run** `npm run type-check` and `npm run build`; expected: both pass.

### Task 5: 管理员工作台

**Files:**
- Modify: `yu-picture-frontend/src/pages/admin/UserManagePage.vue`
- Modify: `yu-picture-frontend/src/pages/admin/PictureManagePage.vue`
- Modify: `yu-picture-frontend/src/pages/admin/SpaceManagePage.vue`
- Modify: `yu-picture-frontend/src/pages/admin/SpaceUserManagePage.vue`

**Interfaces:**
- Consumes: Task 1 workspace shell and existing table/query/action handlers.
- Produces: shared admin page heading, filter panel, status labels, table, pagination and action layout; no endpoint or role changes.

- [x] **Step 1: Add consistent page headings and filter panels** to user, picture and space management while retaining each page’s current search parameters.
- [x] **Step 2: Restyle table cells and status tags** for images, roles, review states, space type and quota data; keep existing data formatting.
- [x] **Step 3: Restyle action groups and delete/review controls** without changing visibility checks or handler calls.
- [ ] **Step 4: At 360px, verify** admin page chrome stays within viewport and wide tables use local horizontal scroll.
- [x] **Step 5: Run** `npm run type-check` and `npm run build`; expected: both pass.

### Task 6: 全站一致性与最终验收

**Files:**
- Inspect all files under `yu-picture-frontend/src`, `public`, `index.html`, `README.md`, and package metadata.
- Modify: `yu-picture-frontend/src/views/AboutView.vue` as a compact brand/about page using the shared workspace shell.
- Delete unused Vue starter files after confirming no imports: `src/views/HomeView.vue`, `src/components/HelloWorld.vue`, `src/components/TheWelcome.vue`, `src/components/WelcomeItem.vue`, and `src/components/icons/IconCommunity.vue`, `IconDocumentation.vue`, `IconEcosystem.vue`, `IconSupport.vue`, `IconTooling.vue`.

**Interfaces:**
- Consumes: all previous tasks.
- Produces: complete brand cleanup and a consistent responsive front end across every registered route.

- [x] **Step 1: Search for old product names, links and assets** with the Task 1 `rg` command and inspect all remaining matches; remove only frontend product/attribution references, not API identifiers.
- [x] **Step 2: Review every registered route** from `src/router/index.ts` and confirm a deliberate layout exists; `/about` uses the brand/about page and no route renders the deleted Vue starter screens.
- [x] **Step 3: Review loading, empty, failure and permission states** on gallery, detail, upload, space, analysis and admin screens; no dummy content or test records.
- [ ] **Step 4: Keyboard-check** header, search, forms, menus and dialogs; focus remains visible and reduced-motion preference suppresses nonessential transitions.
- [x] **Step 5: Run** `npm run type-check` and `npm run build`; expected: clean type check and production bundle.
- [ ] **Step 6: Browser-check** home, login, detail, workspace, analysis and admin screen at desktop and mobile widths; capture screenshots for review.
- [x] **Step 7: Leave the browser on the redesigned home page** after the review; report any authenticated workflow that remains unverified because no real login was available.
