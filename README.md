import geopandas as gpd
import pandas as pd
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
from matplotlib.colors import ListedColormap, BoundaryNorm
from matplotlib.cm import ScalarMappable


csv_path = r"G:\Qurbani\qurbani_statistics_2023_with_category.csv"
shapefile_path = r"G:\Qurbani\world-administrative-boundaries"


qurbani_df = pd.read_csv(csv_path)
qurbani_df.columns = qurbani_df.columns.str.strip().str.lower()
qurbani_df.rename(columns={'country': 'Country', 'category_label': 'Category'}, inplace=True)
qurbani_df['Country'] = qurbani_df['Country'].str.strip().str.lower()

qurbani_df['Category'] = qurbani_df['Category'].replace('Unknown', 'Very Low')


world_gdf = gpd.read_file(shapefile_path)
world_gdf['name'] = world_gdf['name'].str.strip().str.lower()

merged_gdf = world_gdf.merge(qurbani_df, how='left', left_on='name', right_on='Country')
merged_gdf['Category'] = merged_gdf['Category'].fillna('Very Low')


merged_gdf = merged_gdf[merged_gdf['Category'] != 'Medium']


category_order = ['Very Low', 'Low', 'High', 'Very High']
category_colors = {
   'Very Low': '#F4D03F',     # Vibrant Yellow
   'Low': '#45B39D',          # Cyan
   'High': '#AF7AC5',         # Purple
   'Very High': '#E74C3C',    # Bright Red
}
cmap = ListedColormap([category_colors[cat] for cat in category_order])
norm = BoundaryNorm(range(len(category_order)+1), cmap.N)


fig, ax = plt.subplots(figsize=(18, 10))


merged_gdf.plot(
   column=merged_gdf['Category'].map({k: i for i, k in enumerate(category_order)}),
   cmap=cmap,
   linewidth=0.2,
   edgecolor='none',
   ax=ax
)


ax.axis('off')
ax.set_title("Global Qurbani Statistics (2000-2025)", fontsize=20)


sm = ScalarMappable(cmap=cmap, norm=norm)
sm.set_array([])

cbar = fig.colorbar(
   sm,
   ax=ax,
   orientation='vertical',
   fraction=0.025,
   pad=0.02,
   shrink=0.75,
   ticks=[]
)


cbar.ax.set_yticks([i + 0.5 for i in range(len(category_order))])
cbar.ax.set_yticklabels(category_order, fontsize=10)
cbar.set_label("Qurbani Category", fontsize=12)

plt.tight_layout()
plt.savefig("qurbani_world_map_medium.png", dpi=300)
print("Map saved as qurbani_world_map_medium.png")
