<script>
	import { createEventDispatcher, getContext } from 'svelte';
	import Icon from '../icon/Icon.svelte';
	import Page from './Page.svelte';
	import SecondPageNext from './SecondPageNext.svelte';
	import SecondPagePre from './SecondPagePre.svelte';
	import zh_CN from '../../lang/zh_CN';

	// 定义事件派发器
	// Define event dispatcher
	const dispatch = createEventDispatcher();

	// 当前语言
	// current language
	const currentLang = getContext('STDF_lang') || zh_CN;
	const commonLang = currentLang.common;
	const paginationLang = currentLang.pagination;

	/**
	 * 总条数
	 * total
	 * @type {number}
	 * @default 0
	 */
	export let total = 0;

	/**
	 * 每页条数
	 * pageSize
	 * @type {number}
	 * @default 10
	 */
	export let pageSize = 10;

	/**
	 * 当前页
	 * current
	 * @type {number}
	 * @default 1
	 */
	export let current = 1;

	/**
	 * 最大显示页码数
	 * max show page
	 * @type {5|7|9|11}
	 * @default 7
	 */
	export let maxShowPage = 7;

	/**
	 * 圆角
	 * radius
	 * @type {'base'|'md'|'lg'|'xl'|'full'|'none'}
	 * @default 'md'
	 */
	export let radius = 'md';

	/**
	 * 高亮页码类型
	 * highlight page type
	 * @type {'border'|'block'|'bold'}
	 * @default 'bold'
	 */
	export let type = 'bold';

	/**
	 * 省略页码列数
	 * second level page column
	 * @type {number}
	 * @default 3
	 */
	export let pageCol = 3;

	/**
	 * 是否显示后省略号的省略页码
	 * show second level page in next ellipsis
	 * @type {boolean}
	 * @default false
	 */
	export let showNextOmitPage = false;

	/**
	 * 是否显示前省略号的省略页码
	 * show second level page in pre ellipsis
	 * @type {boolean}
	 * @default false
	 */
	export let showPreOmitPage = false;

	/**
	 * 注入CSS
	 * Inject CSS
	 * @type {string}
	 * @default ''
	 */
	export let injClass = '';

	/**
	 * 无数据文本
	 * No data text
	 * @type {string}
	 * @default Current language common.noData
	 */
	export let noDataText = commonLang.noData;

	/**
	 * 仅一页文本
	 * only one page text
	 * @type {string}
	 * @default Current language pagination.defaultOnlyOnePage
	 */
	export let onePageText = paginationLang.defaultOnlyOnePage;

	/**
	 * 连续模式
	 * continuous mode
	 * @type {boolean}
	 * @default false
	 */
	export let continuous = false;

	// 总页数
	// totalPage
	const totalPage = Math.ceil(total / pageSize);

	// 当前页码大于等于 maxShowPage-3 时，不显示前面的省略号
	// current >= maxShowPage-3, not show pre ellipsis
	const showPreEllipsis = current > maxShowPage - 2;

	// 当前页码小于等于 totalPage - (maxShowPage-3) 时，不显示后面的省略号
	// current <= totalPage - (maxShowPage-3), not show next ellipsis
	const showNextEllipsis = current <= maxShowPage - 2 || current <= totalPage - (maxShowPage - 3);

	// 当显示前面的省略号时，中间显示的页码数 middleShowPage 个数为 maxShowPage - 4，内容为当前页码（maxShowPage为5）或者当前页码和前后一项（maxShowPage为7）或者当前页码和前后两项（maxShowPage为9）
	// when show pre ellipsis, middleShowPage length is maxShowPage - 4, content is current page (maxShowPage is 5) or current page and pre/next one (maxShowPage is 7) or current page and pre/next two (maxShowPage is 9)
	let middleShowPage = [];

	// 前省略号内页码数组
	// pre ellipsis page array
	let preEllipsisPages = [];

	// 后省略号内页码数组
	// next ellipsis page array
	let nextEllipsisPages = [];

	// 当页码变化时对一系列数据动态计算
	// Dynamic calculation of a series of data when the page number changes
	$: {
		if (showPreEllipsis && showNextEllipsis) {
			if (maxShowPage === 5) {
				middleShowPage = [current];
			} else if (maxShowPage === 7) {
				middleShowPage = [current - 1, current, current + 1];
			} else if (maxShowPage === 9) {
				middleShowPage = [current - 2, current - 1, current, current + 1, current + 2];
			} else if (maxShowPage === 11) {
				middleShowPage = [current - 3, current - 2, current - 1, current, current + 1, current + 2, current + 3];
			}
		} else {
			middleShowPage = [];
		}
		// 当仅显示后省略号时
		// when only show next ellipsis
		if (!showPreEllipsis && showNextEllipsis) {
			// 此时 nextEllipsisPages 为 maxShowPage - 2 到 总页码数 - 1
			// nextEllipsisPages is maxShowPage - 2 to total page - 1
			nextEllipsisPages = Array.from({ length: totalPage }, (v, k) => k + 1).slice(maxShowPage - 2, totalPage - 1);
		}
		// 当仅显示前省略号时
		// when only show pre ellipsis
		if (showPreEllipsis && !showNextEllipsis) {
			// 此时 preEllipsisPages 为 2 到 总页码数 - (maxShowPage-2)
			// preEllipsisPages is 2 to total page - (maxShowPage-2)
			preEllipsisPages = Array.from({ length: totalPage }, (v, k) => k + 1).slice(1, totalPage - (maxShowPage - 2));
		}
		// 当显示前后省略号时
		// when show pre and next ellipsis
		if (showPreEllipsis && showNextEllipsis) {
			// 此时 nextEllipsisPages 为 middleShowPage 最后一项 +1 ，到总页码数组的最后第二项
			// nextEllipsisPages is middleShowPage last item +1, to total page array last second item
			nextEllipsisPages = Array.from({ length: totalPage }, (v, k) => k + 1).slice(
				middleShowPage[middleShowPage.length - 1],
				totalPage - 1,
			);
			// 此时 preEllipsisPages 为 2 到 middleShowPage 第一项 -1
			// preEllipsisPages is 2 to middleShowPage first item -1
			preEllipsisPages = Array.from({ length: totalPage }, (v, k) => k + 1).slice(1, middleShowPage[0] - 1);
		}
	}

	// 特殊情况处理
	// Special case handling
	if (totalPage <= maxShowPage) {
		showNextOmitPage = false;
	}

	// 点击后省略号事件
	// click next ellipsis event
	const clickNextEllipsisFunc = () => {
		if (nextEllipsisPages.length > 0) {
			showNextOmitPage = !showNextOmitPage;
		}
	};

	// 点击前省略号事件
	// click pre ellipsis event
	const clickPreEllipsisFunc = () => {
		if (preEllipsisPages.length > 0) {
			showPreOmitPage = !showPreOmitPage;
		}
	};

	// 页码改变的回调，参数是改变后的页码及每页条数
	// onChange
	const onChange = () => {
		showNextOmitPage = false;
		showPreOmitPage = false;
		dispatch('change', current);
	};

	// 点击下一页
	// click next page
	const nextFunc = () => {
		if (current < totalPage) {
			current++;
			dispatch('next', current);
			onChange();
		}
	};

	// 点击上一页
	// click pre page
	const preFunc = () => {
		if (current > 1) {
			current--;
			dispatch('pre', current);
			onChange();
		}
	};

	// 点击页码
	// click page
	const clickItemFunc = index => {
		current = index;
		onChange();
	};

	// 点击省略页码事件
	// click second page item event
	const clickSecondPageItemFunc = e => {
		current = e.detail;
		showNextOmitPage = false;
		showPreOmitPage = false;
		onChange();
	};

	// 类型样式
	// type class
	const typeClass = {
		border: 'border-primary dark:border-dark text-primary dark:text-dark',
		block: 'bg-primary text-white dark:bg-dark dark:text-black',
		bold: 'font-bold text-primary dark:text-dark border-transparent',
	};

	// 圆角样式
	// radius class
	const radiusClass = { base: 'rounded', md: 'rounded-md', lg: 'rounded-lg', xl: 'rounded-xl', full: 'rounded-full', none: 'rounded-none' };
</script>

<div class="py-1 bg-white dark:bg-black flex justify-between text-center text-sm relative {injClass}">
	<!-- svelte-ignore a11y-click-events-have-key-events -->
	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<div
		class="flex-1 py-2 border border-transparent transition-all {current > 1
			? 'text-primary dark:text-dark'
			: 'text-primary/30 dark:text-dark/30'} {radiusClass[radius] || radiusClass.base} active:scale-75"
		on:click={preFunc}
	>
		<Icon name="ri-arrow-left-s-line" size={18} />
	</div>
	{#if totalPage === 0}
		<div class="flex-1 py-2 border border-transparent">{noDataText}</div>
	{:else if totalPage === 1}
		<div class="flex-1 py-2 border border-transparent">{onePageText}</div>
	{:else if totalPage > 1 && totalPage <= maxShowPage}
		<!-- eslint-disable-next-line no-unused-vars -->
		{#each new Array(totalPage) as item, index}
			<Page active={current === index + 1} {type} {radius} on:click={() => !continuous && clickItemFunc(index + 1)}>
				{index + 1}
			</Page>
		{/each}
	{:else}
		<Page active={current === 1} {type} {radius} on:click={() => !continuous && clickItemFunc(1)}>1</Page>
		{#if showPreEllipsis}
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-static-element-interactions -->
			<div
				class="flex-1 py-2 border {showPreOmitPage
					? typeClass[type] || typeClass.border
					: 'border-transparent' + (type === 'bold' ? ' opacity-50' : '')} {radiusClass[radius] || radiusClass.base}"
				on:click={!continuous && clickPreEllipsisFunc}
			>
				{#if type === 'bold' && showPreOmitPage}
					<Icon name="ri-more-fill" size={18} />
				{:else}
					<Icon name="ri-more-line" size={18} />
				{/if}
			</div>
		{/if}
		{#if !showPreEllipsis && current <= maxShowPage - 1}
			<!-- eslint-disable-next-line no-unused-vars -->
			{#each new Array(maxShowPage - 3) as item, index}
				<Page active={current === index + 2} {type} {radius} on:click={() => !continuous && clickItemFunc(index + 2)}>
					{index + 2}
				</Page>
			{/each}
		{/if}
		{#if middleShowPage.length > 0}
			{#each middleShowPage as item, index}
				<Page active={index === (middleShowPage.length - 1) / 2} {type} {radius} on:click={() => !continuous && clickItemFunc(item)}>
					{item}
				</Page>
			{/each}
		{/if}
		{#if !showNextEllipsis && current > totalPage - (maxShowPage - 3)}
			<!-- eslint-disable-next-line no-unused-vars -->
			{#each new Array(maxShowPage - 3) as item, index}
				<Page
					active={current === totalPage + index + 3 - maxShowPage}
					{type}
					{radius}
					on:click={() => !continuous && clickItemFunc(totalPage + index + 3 - maxShowPage)}
				>
					{totalPage + index + 3 - maxShowPage}
				</Page>
			{/each}
		{/if}
		{#if showNextEllipsis}
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-static-element-interactions -->
			<div
				class="flex-1 py-2 border {showNextOmitPage
					? typeClass[type] || typeClass.border
					: 'border-transparent' + (type === 'bold' ? ' opacity-50' : '')} {radiusClass[radius] || radiusClass.base}"
				on:click={!continuous && clickNextEllipsisFunc}
			>
				{#if type === 'bold' && showNextOmitPage}
					<Icon name="ri-more-fill" size={18} />
				{:else}
					<Icon name="ri-more-line" size={18} />
				{/if}
			</div>
		{/if}
		<Page active={current === totalPage} {type} {radius} on:click={() => !continuous && clickItemFunc(totalPage)}>{totalPage}</Page>
	{/if}
	<!-- svelte-ignore a11y-click-events-have-key-events -->
	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<div
		class="flex-1 py-2 border border-transparent transition-all {current < totalPage
			? 'text-primary dark:text-dark'
			: 'text-primary/30 dark:text-dark/30'} {radiusClass[radius] || radiusClass.base} active:scale-75"
		on:click={nextFunc}
	>
		<Icon name="ri-arrow-right-s-line" size={20} />
	</div>
	{#if showNextOmitPage}
		<SecondPageNext {pageCol} Pages={nextEllipsisPages} {type} {radius} on:clickItem={clickSecondPageItemFunc} {maxShowPage} />
	{/if}
	{#if showPreOmitPage}
		<SecondPagePre {pageCol} Pages={preEllipsisPages} {type} {radius} on:clickItem={clickSecondPageItemFunc} {maxShowPage} />
	{/if}
</div>
