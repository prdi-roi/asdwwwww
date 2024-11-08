# asdwwwww

```
package com.prodia.mobileandroid.ui.cdm.landing.content

import androidx.annotation.DrawableRes
import androidx.compose.foundation.Image
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.defaultMinSize
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.heightIn
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.grid.GridCells
import androidx.compose.foundation.lazy.grid.LazyVerticalGrid
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.Icon
import androidx.compose.material.IconButton
import androidx.compose.material.Text
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.ArrowBack
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.alpha
import androidx.compose.ui.draw.clip
import androidx.compose.ui.draw.shadow
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.vector.ImageVector
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.platform.LocalConfiguration
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalDensity
import androidx.compose.ui.res.colorResource
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.res.vectorResource
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.Dp
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.core.os.bundleOf
import androidx.fragment.app.FragmentManager
import androidx.lifecycle.viewmodel.compose.viewModel
import androidx.navigation.NavController
import coil.compose.rememberAsyncImagePainter
import coil.request.ImageRequest
import coil.size.Size
import com.google.accompanist.pager.ExperimentalPagerApi
import com.google.accompanist.pager.rememberPagerState
import com.prodia.mobileandroid.R
import com.prodia.mobileandroid.design.swiperefresh.ProdiaSwipeRefresh
import com.prodia.mobileandroid.design.text.poppinsFamily
import com.prodia.mobileandroid.design.toolbar.TopBar
import com.prodia.mobileandroid.domain.cdm.entity.cdmresult.CdmResultEntity
import com.prodia.mobileandroid.domain.cdm.entity.cdmresult.CdmTodaysLogEntity
import com.prodia.mobileandroid.domain.cdm.entity.cdmresult.MenuEntity
import com.prodia.mobileandroid.domain.cdm.entity.cdmresult.MenuType
import com.prodia.mobileandroid.domain.cdm.entity.cdmresult.SpecialStatusCategory
import com.prodia.mobileandroid.domain.cdm.entity.cdmresult.SpecialStatusValue
import com.prodia.mobileandroid.domain.healthplanv2.usecase.model.HealthPlanProduct
import com.prodia.mobileandroid.domain.healthscorev2.entity.AssessmentRisk
import com.prodia.mobileandroid.domain.healthscorev2.entity.RiskTypeCDM
import com.prodia.mobileandroid.ui.cdm.landing.AssessmentInsightSectionUiState
import com.prodia.mobileandroid.ui.cdm.landing.CdmLandingFragment
import com.prodia.mobileandroid.ui.cdm.landing.CdmLandingViewModel
import com.prodia.mobileandroid.ui.cdm.landing.LoadingView
import com.prodia.mobileandroid.ui.cdm.landing.assessment_result.CdmAssessmentResultFragment
import com.prodia.mobileandroid.ui.cdm.landing.components.BadgeTopBar
import com.prodia.mobileandroid.ui.healthscore.HealthAssessmentFragment
import com.prodia.mobileandroid.ui.healthscore.assessment.AssessmentType
import com.prodia.mobileandroid.ui.healthscore.utils.checkPrerequisite
import com.prodia.mobileandroid.utils.AndroidDeviceUtil
import com.prodia.mobileandroid.utils.AppConstants.DIABETES
import com.prodia.mobileandroid.utils.AppVersionUtils
import com.prodia.mobileandroid.utils.collectAsStateLifecycleAware
import com.prodia.mobileandroid.utils.convertToTitleCase
import java.util.Locale
import com.prodia.mobileandroid.design.R as designR

@OptIn(ExperimentalPagerApi::class)
@Composable
fun CdmLandingContentScreen(
    navController: NavController? = null,
    cdmResult: List<CdmResultEntity> = listOf(),
    codeAssessment: String = DIABETES,
    cdmViewModel: CdmLandingViewModel = viewModel(),
    childFragmentManager: FragmentManager
) {
    val uiStateResultAssessment =
        cdmViewModel.uiState.collectAsStateLifecycleAware().value
    val isCDMLandingAlreadyAccessed =
        cdmViewModel.isCDMLandingEventEmitted.collectAsStateLifecycleAware().value

    val cdmResultData = cdmResult.firstOrNull {
        it.category == codeAssessment.uppercase(Locale.ROOT)
    } ?: CdmResultEntity()
    val isUnknownType =
        RiskTypeCDM.getValueOf(cdmResultData.riskType.orEmpty()) == RiskTypeCDM.UNKNOWN

    val imageHeaders = mutableListOf<String>()
    cdmResultData.headerContentList?.sortedBy { it.order }?.forEach {
        imageHeaders.add(it.imgUrl.orEmpty())
    }

    val customSystemMessage = cdmViewModel.customSystemMessage.collectAsStateLifecycleAware().value

    val pagerState = rememberPagerState()
    val scrollState = rememberScrollState()
    val maxScroll = 120.dp.value * LocalDensity.current.density
    val transparency = (scrollState.value / maxScroll).coerceIn(0f, 1f)

    val specialStatus = cdmResultData.specialStatus?.firstOrNull()
    val isDiabetesOnBoarding =
        specialStatus?.statusCategory == SpecialStatusCategory.DIABETES && specialStatus.statusValue == SpecialStatusValue.DIABETES_MELLITUS

    val getAssessmentResult = {
        if (isUnknownType.not()) {
            cdmViewModel.getHealthScoreResult(codeAssessment)
        } else {
            cdmViewModel.updateUiState(
                AssessmentInsightSectionUiState.Hidden
            )
        }
    }

    val riskType = RiskTypeCDM.getValueOf(cdmResultData.riskType.orEmpty()).name
    val stage = riskType.getHeaderState()
    val badgeCountNotification = 1

    LaunchedEffect(Unit) {
        getAssessmentResult()
        val riskTypeCDM = RiskTypeCDM.getValueOf(cdmResultData.riskType.orEmpty())
        if (!isCDMLandingAlreadyAccessed) {
            cdmViewModel.trackCdmLandingPageAccessedByLoginUser(
                riskTypeCDM = riskTypeCDM,
                appVersion = AppVersionUtils.getVersionName(),
                osVersion = AndroidDeviceUtil.getOsNameVersion(),
            )
        }
    }

    when (uiStateResultAssessment) {
        AssessmentInsightSectionUiState.Loading -> {
            Column {
                TopBar(
                    modifier = Modifier.padding(top = 32.dp),
                    backgroundColor = com.prodia.mobileandroid.design.R.color.primary_white,
                    elevation = 0.dp,
                    onBackClick = { navController?.popBackStack() },
                    transparency = 0f
                )
                LoadingView()
            }
        }

        else -> {
            ProdiaSwipeRefresh(
                onRefresh = {
                    cdmViewModel.getCdmResult()
                    getAssessmentResult()
                }
            ) {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                ) {

                    /**
                     * Content Section
                     */
                    val maxHeight = calculateContentHeight(
                        sizeOfElement = cdmResultData.menuList?.size ?: 0,
                    )
                    Column(
                        modifier = Modifier
                            .fillMaxSize()
                            .verticalScroll(state = scrollState)
                            .fillMaxWidth()
                            .heightIn(max = maxHeight)
                    ) {
                        BackgroundHeader(
                            backgroundHeader = stage.background,
                            headerCaptionImage = stage.iconCaption,
                            headerCaption = stringResource(id = stage.caption),
                        )

                        Text(
                            modifier = Modifier
                                .padding(top = 24.dp, start = 16.dp),
                            text = "Today's Log",
                            fontFamily = poppinsFamily,
                            fontSize = 18.sp,
                            fontWeight = FontWeight.SemiBold,
                            lineHeight = 24.sp,
                            color = colorResource(id = designR.color.text_color_black),
                        )
                        TodaysLogUI(logs = dummyLogs())

                        val bannerUrl = "https://github.com/user-attachments/assets/e29576a3-8f83-4e27-ad48-bc4fbacfd916"
                        if (bannerUrl.isNotEmpty()) BannerUI(imageUrl = bannerUrl)

                        Text(
                            modifier = Modifier
                                .padding(top = 24.dp, start = 16.dp),
                            text = "Services For You",
                            fontFamily = poppinsFamily,
                            fontSize = 18.sp,
                            fontWeight = FontWeight.SemiBold,
                            lineHeight = 24.sp,
                            color = colorResource(id = designR.color.text_color_black),
                        )
                        cdmResultData.menuList?.let { services ->
                            ServicesUI(
                                services = services,
                            ) { index ->
                                onClickCard(
                                    menu = services[index],
                                    navController = navController,
                                    isUnknownType = isUnknownType,
                                    codeAssessment = codeAssessment,
                                    cdmResultData = cdmResultData,
                                    childFragmentManager = childFragmentManager,
                                    cdmViewModel = cdmViewModel,
                                    customSystemMessage = customSystemMessage,
                                    riskType = RiskTypeCDM.getValueOf(cdmResultData.riskType.orEmpty())
                                )
                            }
                        }
                    }

                    /**
                     * Header Section
                     */
                    Box {
                        Column(
                            modifier = Modifier
                                .fillMaxWidth()
                                .alpha(if (transparency > 0.5f) 1f else 0f)
                                .background(colorResource(id = com.prodia.mobileandroid.design.R.color.primary_white))
                                .height(32.dp)
                        ) {}
                        HeaderUI(
                            transparency = transparency,
                            stageLevel = stringResource(id = stage.level),
                            stageLevelColors = stage.gradientColors,
                            badgeCountNotification = badgeCountNotification,
                            onBack = { navController?.popBackStack() },
                            onNotificationClicked = { }
                        )
                        Spacer(
                            modifier = Modifier
                                .fillMaxWidth()
                                .height(1.dp)
                                .alpha(if (transparency > 0.5f) 1f else 0f)
                                .shadow(1.dp)
                                .align(Alignment.BottomCenter)
                        )
                    }
                }
            }
        }
    }
}

@Composable
private fun calculateContentHeight(
    sizeOfElement: Int,
): Dp {
    val screenHeight = LocalConfiguration.current.screenHeightDp.dp
    val defaultServiceItemMinHeight = 160
    val minHeightOfServiceItem = defaultServiceItemMinHeight.times(sizeOfElement)
    return screenHeight + minHeightOfServiceItem.dp
}

@Composable
private fun BannerUI(
    imageUrl: String,
) {
    Image(
        modifier = Modifier
            .padding(horizontal = 16.dp, vertical = 24.dp)
            .fillMaxWidth()
            .defaultMinSize(minHeight = 158.dp),
        painter = rememberAsyncImagePainter(
            model = ImageRequest.Builder(LocalContext.current)
                .data(imageUrl)
                .placeholder(R.drawable.ic_placeholder)
                .error(R.drawable.ic_placeholder)
                .build()
        ),
        contentDescription = "Banner",
        contentScale = ContentScale.FillBounds
    )
}

@Preview(showSystemUi = true, showBackground = true)
@Composable
private fun BannerUIPreview() {
    BannerUI("imageUrl")
}

@Composable
private fun BackgroundHeader(
    @DrawableRes backgroundHeader: Int,
    @DrawableRes headerCaptionImage: Int,
    headerCaption: String,
) {
    Box(
        modifier = Modifier
            .height(220.dp)
    ) {
        Image(
            modifier = Modifier.fillMaxSize(),
            imageVector = ImageVector.vectorResource(id = backgroundHeader),
            contentScale = ContentScale.FillBounds,
            contentDescription = null,
        )

        Row(
            modifier = Modifier
                .padding(start = 16.dp, end = 16.dp, bottom = 16.dp)
                .fillMaxSize(),
            verticalAlignment = Alignment.Bottom
        ) {
            Text(
                modifier = Modifier
                    .weight(2f)
                    .padding(bottom = 16.dp, end = 8.dp),
                text = headerCaption,
                fontFamily = poppinsFamily,
                fontSize = 20.sp,
                fontWeight = FontWeight.SemiBold,
                lineHeight = 26.sp,
                color = colorResource(id = designR.color.primary_black),
            )
            Image(
                modifier = Modifier
                    .defaultMinSize(minWidth = 113.dp, minHeight = 95.dp)
                    .weight(1f),
                imageVector = ImageVector.vectorResource(id = headerCaptionImage),
                contentDescription = "Landing Header",
                contentScale = ContentScale.FillBounds,
            )
        }
    }
}

@Preview(showBackground = true, showSystemUi = true)
@Composable
private fun BackgroundHeaderPreview() {
    Column(
        Modifier.fillMaxSize()
    ) {
        BackgroundHeader(
            backgroundHeader = R.drawable.ic_header_stage_0,
            headerCaptionImage = R.drawable.ic_header_landing_1,
            headerCaption = "Diabetes-Free Path, Start with Prevention"
        )
    }
}

@Composable
private fun ServicesUI(
    services: List<MenuEntity>,
    onServiceClicked: (index: Int) -> Unit,
) {
    LazyColumn(
        modifier = Modifier
            .fillMaxWidth()
            .padding(all = 16.dp),
        userScrollEnabled = false,
    ) {
        items(services.size) { index ->
            CdmLandingContentItemCard(
                services[index].title.orEmpty(),
                services[index].desc.orEmpty(),
                services[index].img.orEmpty(),
            ) { onServiceClicked.invoke(index) }
        }
    }
}

@Preview(showBackground = true, showSystemUi = true)
@Composable
private fun ServicesUIPreview() {
    ServicesUI(
        services = listOf(
            MenuEntity(
                title = "Start & Maintain Healthy Lifestyle",
                desc = "Easier & more effective with a plan that’s personalized for you",
                img = "imageUrl",
                order = null,
                type = null,
            ),
            MenuEntity(
                title = "Consult with Doctor",
                desc = "Ask & confirm diabetes-related condition to doctors",
                img = "imageUrl",
                order = null,
                type = null,
            ),
            MenuEntity(
                title = "Take Diabetes Lab Test",
                desc = "Take lab test to confirm your diabetes risk for sure",
                img = "imageUrl",
                order = null,
                type = null,
            ),
        ),
    ) { }
}

@Composable
private fun TodaysLogUI(
    logs: List<CdmTodaysLogEntity>,
) {
    LazyVerticalGrid(
        modifier = Modifier
            .padding(horizontal = 16.dp)
            .padding(top = 16.dp)
            .fillMaxWidth(),
        columns = GridCells.Fixed(2),
        userScrollEnabled = false,
    ) {
        items(logs.size) { index ->
            LogUI(log = logs[index])
        }
    }
}

@Composable
private fun LogUI(log: CdmTodaysLogEntity) {
    Column(
        modifier = Modifier
            .height(78.dp)
            .padding(all = 4.dp)
            .clip(RoundedCornerShape(16.dp))
            .background(colorResource(id = designR.color.sys_grey_1))
            .padding(horizontal = 16.dp),
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = log.title,
            fontFamily = poppinsFamily,
            fontSize = 12.sp,
            fontWeight = FontWeight.Normal,
            lineHeight = 18.sp,
            color = colorResource(id = designR.color.text_color_black),
        )
        Row(
            modifier = Modifier
                .padding(top = 4.dp),
            verticalAlignment = Alignment.CenterVertically,
        ) {
            Image(
                modifier = Modifier
                    .size(20.dp)
                    .padding(end = 8.dp),
                imageVector = ImageVector.vectorResource(id = log.image),
                contentDescription = "${log.title} Icon",
            )
            Text(
                text = log.content,
                fontFamily = poppinsFamily,
                fontSize = 14.sp,
                fontWeight = FontWeight.Medium,
                lineHeight = 20.sp,
                color = colorResource(id = designR.color.text_color_black),
            )
        }
    }
}

@Preview(showBackground = true, showSystemUi = true)
@Composable
private fun TodaysLogPreview() {
    TodaysLogUI(dummyLogs())
}

private fun dummyLogs() = listOf(
    CdmTodaysLogEntity(
        image = R.drawable.ic_address_office,
        title = "Body Measurement",
        content = "25.9 kg/m2",
    ),
    CdmTodaysLogEntity(
        image = R.drawable.ic_complete_survey,
        title = "Meal Log",
        content = "330 kcal",
    ),
    CdmTodaysLogEntity(
        image = R.drawable.ic_book_lab_test,
        title = "Blood Glucose",
        content = "68 mg/dL",
    ),
    CdmTodaysLogEntity(
        image = R.drawable.ic_breakfast,
        title = "Blood Pressure",
        content = "110/80 mmHg",
    ),
    CdmTodaysLogEntity(
        image = R.drawable.ic_dietary,
        title = "Supplements & Meds",
        content = "0/3 Taken",
    )
)

@Composable
private fun HeaderUI(
    stageLevel: String,
    stageLevelColors: List<Int>,
    badgeCountNotification: Int,
    transparency: Float,
    onBack: () -> Unit,
    onNotificationClicked: () -> Unit,
) {
    Row(
        modifier = Modifier
            .background(
                if (transparency > 0.5f)
                    colorResource(id = com.prodia.mobileandroid.design.R.color.primary_white)
                else Color.Unspecified
            )
            .fillMaxWidth()
            .padding(top = 40.dp, end = 16.dp, bottom = 12.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        IconButton(
            onClick = { onBack.invoke() }
        ) {
            Icon(Icons.Filled.ArrowBack, "backIcon")
        }
        Column(
            modifier = Modifier.fillMaxWidth(),
            horizontalAlignment = Alignment.End
        ) {
            BadgeTopBar(
                stage = stageLevel,
                stageLevelColors = stageLevelColors,
                badgeCount = badgeCountNotification
            ) {
                onNotificationClicked.invoke()
            }
        }
    }
}

@Preview
@Composable
private fun HeaderUIPreview() {
    HeaderUI(
        stageLevel = "Stage 0",
        stageLevelColors = listOf(R.color.cdm_stage_0_start, R.color.cdm_stage_0_end),
        badgeCountNotification = 20,
        transparency = 1f,
        onBack = { },
        onNotificationClicked = {}
    )
}

@Composable
private fun ToolbarLanding(
    transparency: Float,
    isDiabetesOnBoarding: Boolean,
    isUnknownType: Boolean,
    codeAssessment: String,
    uiStateResultAssessment: AssessmentInsightSectionUiState,
    cdmViewModel: CdmLandingViewModel,
    navController: NavController?
) {
    Row(
        modifier = Modifier
            .padding(top = 32.dp)
            .background(
                if (transparency > 0.5f)
                    colorResource(id = com.prodia.mobileandroid.design.R.color.primary_white)
                else Color.Unspecified
            )
            .fillMaxWidth()
            .padding(end = 16.dp, bottom = 12.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        IconButton(
            onClick = {
                navController?.popBackStack()
            }
        ) {
            Icon(Icons.Filled.ArrowBack, "backIcon")
        }
        Column(
            modifier = Modifier.fillMaxWidth(),
            horizontalAlignment = Alignment.End
        ) {
            if (isDiabetesOnBoarding) {
                cdmViewModel.setCustomSystemMessage(
                    stringResource(id = R.string.cdm_consultation_system_message_health_profile_diabetes)
                )
                CdmLandingContentBadgeResult(
                    assessmentRisk = AssessmentRisk.HIGH,
                    title = "${stringResource(id = R.string.health_profile)} ",
                    riskTitle = stringResource(id = R.string.has_diabetes),
                    isClickable = false
                ) {}
            } else if (isUnknownType.not()) {
                when (uiStateResultAssessment) {
                    is AssessmentInsightSectionUiState.Insight -> {
                        cdmViewModel.setCustomSystemMessage(
                            stringResource(
                                id = R.string.cdm_consultation_system_message_diabetes_risk,
                                uiStateResultAssessment.result.riskTitle.convertToTitleCase()
                            )
                        )
                        CdmLandingContentBadgeResult(
                            resultAssessment = uiStateResultAssessment.result,
                            assessmentRisk = uiStateResultAssessment.assessmentRisk
                        ) {
                            val bundle = CdmAssessmentResultFragment.getBundle(
                                code = codeAssessment,
                                assessmentType = AssessmentType.DISEASE_RISK.enumString,
                                isFromAssessmentSurvey = false
                            )
                            navController?.navigate(
                                R.id.cdmAssessmentResultFragment,
                                bundle
                            )
                        }
                    }

                    else -> Unit
                }
            }
        }
    }
}

private fun onClickCard(
    menu: MenuEntity,
    navController: NavController? = null,
    isUnknownType: Boolean,
    codeAssessment: String,
    cdmResultData: CdmResultEntity,
    childFragmentManager: FragmentManager,
    cdmViewModel: CdmLandingViewModel,
    customSystemMessage: String? = null,
    riskType: RiskTypeCDM
) {
    val categoryId = cdmResultData.categoryId.orEmpty()
    val isLowRisk = RiskTypeCDM.getValueOf(cdmResultData.riskType.orEmpty()) == RiskTypeCDM.LOW_RISK
    menu.type?.let { type ->
        val itemHeaderContent = cdmResultData.headerPageContentList?.let { content ->
            content.filter { it.type == type }
        }?.firstOrNull()
        when (MenuType.invoke(type)) {
            MenuType.ASSESSMENT -> {
                navController?.let { nav ->
                    if (isUnknownType) {
                        checkPrerequisite(childFragmentManager, cdmViewModel, true) {
                            HealthAssessmentFragment.navigationToAssessmentSurvey(
                                nav,
                                code = codeAssessment.lowercase(),
                                assessmentType = AssessmentType.DISEASE_RISK.enumString,
                                isFromCDM = true
                            )
                        }
                    } else {
                        val bundle = bundleOf(
                            CdmLandingFragment.HeaderPageContent to itemHeaderContent?.copy(
                                isLowRisk = isLowRisk
                            ),
                            CdmLandingFragment.CategoryId to categoryId
                        )
                        nav.navigate(R.id.lifeStyleCdmFragment, bundle)
                    }
                }
            }

            MenuType.LABTEST -> {
                val bundle = bundleOf(
                    CdmLandingFragment.HeaderPageContent to itemHeaderContent,
                    CdmLandingFragment.CategoryId to categoryId
                )
                navController?.navigate(R.id.cdmLabTestFragment, bundle)
            }

            MenuType.ARTICLE -> {
                val bundle = bundleOf(CdmLandingFragment.HeaderPageContent to itemHeaderContent)
                navController?.navigate(R.id.articleListCdmFragment, bundle)
            }

            MenuType.HEALTHPLAN -> {
                val bundle = bundleOf(
                    CdmLandingFragment.HeaderPageContent to itemHeaderContent,
                    CdmLandingFragment.HealthPlanProduct to HealthPlanProduct(
                        categoryId,
                        riskType.name
                    )
                )
                navController?.navigate(
                    R.id.healthPlanServiceCdmFragment,
                    bundle
                )
            }

            MenuType.CONSULTATION -> {
                val bundle = bundleOf(
                    CdmLandingFragment.HeaderPageContent to itemHeaderContent?.copy(isLowRisk = isLowRisk),
                    CdmLandingFragment.CustomSystemMessage to customSystemMessage
                )
                navController?.navigate(
                    R.id.cdmDoctorConsultationFragment,
                    bundle
                )
            }

            MenuType.HEALTHSHOP -> {
                val bundle = bundleOf(
                    CdmLandingFragment.HeaderPageContent to itemHeaderContent,
                )
                navController?.navigate(
                    R.id.cdmHealthShopServiceFragment,
                    bundle
                )
            }

            else -> {}
        }
    }
}

@Preview
@Composable
fun BadgeLong() {
    Row(
        modifier = Modifier
            .alpha(1f)
            .background(colorResource(id = com.prodia.mobileandroid.design.R.color.primary_white))
            .fillMaxWidth()
            .padding(end = 16.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        IconButton(onClick = { }) {
            Icon(Icons.Filled.ArrowBack, "backIcon")
        }
        Column(
            modifier = Modifier.fillMaxWidth(),
            horizontalAlignment = Alignment.End
        ) {
            CdmLandingContentBadgeResult(
                assessmentRisk = AssessmentRisk.LOW,
                title = "Resiko Diabetes ",
                riskTitle = "Mungkin Di Bawah Rata-rata rata",
                isClickable = false
            ) {}
        }
    }
}

```
