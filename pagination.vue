<template>
	<div :class="[
			{ 'text-left': align === 'left' },
			{ 'text-center': align === 'center' },
			{ 'text-right': align === 'right' },
		]"
		v-if="pages.length > 1"
	>
		<button :class="[
				classes,
				{ 'bg-gray-300': value === 1 },
			]"
			:disabled="value === 1"
			@click="previousPage()"
		>
			<slot name="previous">Prev</slot>
		</button>

		<button :class="[
				classes,
				{ 'bg-blue-500 border-blue-500 text-white': item.page === value }
			]"
			:disabled="item.page === value"
			:key="index"
			v-for="(item, index) in pages"
			@click="switchPage(item.page)"
		>
			{{ item.page }}
		</button>

		<button :class="[
				classes,
				{ 'bg-gray-300': value === total },
			]"
			:disabled="value === total"
			@click="nextPage()"
		>
			<slot name="next">Next</slot>
		</button>
	</div>
</template>

<script>
	export default
	{
		props:
		{
			align:
			{
				type: String,
				default: 'left'
			},

			clickHandler:
			{
				type: Function,
				default: () => {}
			},

			itemsPerPage:
			{
				type: Number,
				default: 10,
			},

			pageRange:
			{
				type: Number,
				default: 1
			},

			totalItems:
			{
				type: Number,
				default: 0,
			},

			updateUrl:
			{
				type: Boolean,
				default: true
			},

			value:
			{
				type: Number,
				default: 1
			}
		},

		data()
		{
			return {
				classes: 'border border-gray-300 border-l-0 font-bold px-2 py-1 text-center text-xs tracking-widest uppercase first:border-l',
			}
		},

		mounted()
		{
			this.switchPage(this.value)
		},

		computed:
		{
			/**
			 * Calculate an array of pages.
			 *
			 * @return array
			 */
			pages()
			{
				const page = this.value
				const range = this.pageRange
				let items = []

				if(this.total <= 4)
				{
					for(let index = 0; index < this.total; index++)
					{
						items.push({ page: index + 1 })
					}
				} else
				{
					items.push({ page: 1 })

					if(page === 1)
					{
						for(let index = 1; index < range + 1; index++)
						{
							items.push({ page: index + 1 })
						}

						items.push({ page: '...' })
					}

					else if(page === this.total)
					{
						items.push({ page: '...' })

						const maxmin = this.total - range
						for(let index = maxmin; index < this.total; index++)
						{
							items.push({ page: index })
						}
					}

					else
					{
						const min = page - range
						const max = page + range

						if(min > 1)
						{
							if(min - 1 !== 1)
							{
								items.push({ page: '...' })
							}

							items.push({ page: min })
						}

						items.push({ page: page })

						if(max < this.total)
						{
							items.push({ page: max })

							if(max + 1 !== this.total)
							{
								items.push({ page: '...' })
							}
						}
					}

					items.push({ page: this.total })
				}

				return items
			},

			/**
			 * The total number of pages.
			 *
			 * @return void
			 */
			total()
			{
				return Math.ceil(this.totalItems / this.itemsPerPage)
			}
		},

		methods:
		{
			nextPage()
			{
				this.switchPage(this.value + 1)
			},

			previousPage()
			{
				this.switchPage(this.value - 1)
			},

			switchPage(page)
			{
				if(this.updateUrl)
				{
					this.$router.push({ query: { page: page }})
				}
				this.$emit('input', page)
				this.clickHandler(page)
			}
		}
	}
</script>